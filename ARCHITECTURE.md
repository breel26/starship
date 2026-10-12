# Starship — Architecture

A resource / exploration / crafting / scheduling game that starts with 1900-era technology and builds toward a starship and travel to other star systems.

**Stack:** TypeScript · Phaser 4 · Zod 4 Mini · Vite · Vitest
**Principle:** data-driven design. All game content (resources, technologies, buildings, workers, equipment, recipes, eras) lives in JSON. Code defines *rules*; JSON defines *content*.
**Working rules:** see CLAUDE.md.

### Dependencies
| Package | Where | Why |
|---|---|---|
| Phaser 4 (pinned 4.2.1) | runtime | rendering, input, scenes. Chosen over Phaser 3, which stopped getting releases after 3.90 (May 2025); v4 also has a GPU tilemap layer that helps large maps on phones |
| Zod 4 (`zod/mini` only) | runtime | validates built-in data and future expansion/mod packs; types come from schemas via `z.infer`. Adds ~10 KB gzipped with English error messages |
| TypeScript, Vite, Vitest | dev | compile, bundle, test |

Nothing else without approval. Versions pinned exactly.

---

## 1. Core idea

Keep the game **simulation** in plain TypeScript with no Phaser dependency. Phaser only **draws** what the simulation says and forwards player input as commands.

Benefits:
- Saving/loading is trivial (state is one serializable object)
- Simulation logic is unit-testable without a browser
- Content changes need no code changes
- Future options open up: offline progress, modding, headless balance testing

---

## 2. Layers

### 2.1 Data (JSON)
Every piece of game content is a JSON file under `/data`, split into one folder per era so files stay small. World-generation content (terrains, planet types, galaxy and fair-start rules) lives in `/data/world`. Nothing about content is hard-coded.

Expansion and mod packs use the same folder format and go through the same validation, so the game is expandable without code changes.

**Everything is expandable, including eras.** Eras, routes, vehicles, buildings and technologies are all data. Adding an era means adding an entry to `eras.json` and a new era folder; nothing in code lists eras by name. Eras are sorted by an `order` number with gaps (100, 200, 300…), so a new era can be slotted between two existing ones (e.g. 250) without renumbering anything or breaking saves.

### 2.2 Registry
At boot:
1. Load all JSON files (built-in, then any packs) and merge them
2. Validate each against a Zod 4 Mini schema
3. Check cross-references (e.g. a recipe input `iron_ore` must exist in `resources.json`; a building's `requiresTech` must exist in `technologies.json`)
4. Index everything by `id` into a read-only registry

Bad data fails loudly at startup, never mid-game.

### 2.3 Simulation
- Pure TS systems run on a **fixed tick: one tick = one 8-hour shift**, 3 ticks per in-game day (see "Time & speed" below)
- World state is a single plain object (`GameState`)
- Systems read the registry + state and produce the next state
- Player actions arrive as **commands** (e.g. `{ type: "build", worldId: "home", buildingId: "coal_furnace", x: 12, y: 8 }`), validated and applied at the start of a tick
- **Every explored world ticks**, whether or not the player is looking at it. Only the visible world is rendered.

Initial systems:
| System | Responsibility |
|---|---|
| production | Runs recipes in buildings, consumes inputs, produces outputs |
| scheduling | Decides which worker goes to which building slot and shift (never how they get there) |
| movement | Works out each worker's commute, which reduces their output for the shift, using the active movement strategy (see §7) |
| research | Progresses technologies, unlocks content (research is global, not per world) |
| prospecting | Reveals mineral deposits when a prospector visits an area |
| logistics | Moves resources between storage and buildings within one world |
| shipping | Moves cargo between worlds; cargo is in transit for a travel time and arrives in the destination world's stockpile |

### Time & speed
- **One tick = one 8-hour shift.** 3 ticks = 1 in-game day.
- **At 1x, a tick takes 2 seconds**, so a day takes 6 seconds and a year about 37 minutes.
- All durations in JSON are written in shifts, e.g. `"durationShifts": 3` for a day-long recipe. Nothing is measured in raw seconds or frames.
- Buildings can run 1, 2 or 3 shifts per day; scheduling assigns crews to shifts.

| Speed | One tick | One day | One year | Ticks per second |
|---|---|---|---|---|
| Paused | — | — | — | 0 |
| 1x | 2 s | 6 s | ~37 min | 0.5 |
| 2x | 1 s | 3 s | ~18 min | 1 |
| 10x | 0.2 s | 0.6 s | ~3.7 min | 5 |
| 100x | 0.02 s | 0.06 s | ~22 s | 50 |
| 1000x | 0.002 s | 0.006 s | ~2.2 s | 500 |

- The tick size never changes with speed; higher speeds just run more ticks per second. Even 1000x (500 ticks/s) is affordable on a phone because each tick covers a whole shift.
- **Frame budget:** if a device can't keep up, the game runs as fast as it can and the speed display shows the actual speed achieved, rather than freezing.
- **Auto-pause options** for key events (research complete, stockpile empty, shipment arrived) so fast speeds don't skip past something important.
- **Smooth rendering:** at low speeds the renderer interpolates progress bars and animations between ticks; at high speeds it redraws less often.
- Speed is a UI setting, not game state; loading a save always starts paused.

### 2.4 Presentation (Phaser)
- Scenes: Boot/Preload, Map (one world at a time, max 12 km in view), WorldOverview (coarse whole-planet map for jumping around), StarMap (choose system and planet), UI overlay, Menus
- Reads `GameState` to render; never mutates it directly
- Sends commands to the sim
- An **event bus** pushes sim events to the UI (`research:complete`, `deposit:discovered`, `building:idle`, …)
- Phaser scale mode handles phone and desktop

---

## 3. Repo layout

```
/data
  eras.json
  economy.json      GORE constants and the value window (starts at 100 ticks)
  /steam            resources.json, buildings.json, recipes.json,
                    technologies.json, workers.json, equipment.json,
                    routes.json, vehicles.json
  /…                one folder per era, same files
  /world            terrains.json, planetTypes.json, galaxy.json, fairStart.json
/src
  main.ts           starts Phaser
  config.ts         tick rate, tile size, constants
  /core             shared types, ids, event bus, rng (no game logic)
  /data
    /schemas        one Zod schema file per content type
    loader.ts       loads + merges era folders and packs
    crossref.ts     checks every id reference resolves
    registry.ts     read-only lookup by id
  /sim
    state.ts  tick.ts  commands.ts
    /systems        production.ts, scheduling.ts, research.ts,
                    prospecting.ts, logistics.ts, shipping.ts, movement.ts
    /movement       swappable travel strategies (see §7):
                    types.ts      the Movement interface
                    distance.ts   straight-line travel time (now)
                    terrain.ts    distance maps (planned upgrade)
  /map              pure logic, no Phaser:
                    noise.ts      seeded noise that wraps seamlessly east–west (our own, no dependency)
                    galaxy.ts     generates star systems and planets from a seed
                    chunks.ts     generates 32×32-tile chunks on demand; cache + discard
                    generate.ts   builds one chunk's tiles from its planet type
                    fairStart.ts  checks/places guaranteed starting resources
                    explored.ts   coarse 1 km exploration grid
                    fog.ts        discovery and visibility
  /render
    /scenes         BootScene, MapScene, WorldOverviewScene, StarMapScene,
                    UIScene, MenuScene
    chunkView.ts    loads/unloads chunk layers near the camera; full art zoomed in,
                    one pre-rendered image per chunk zoomed out
    origin.ts       re-centers render coordinates on the camera (no far-away jitter)
    /sprites        BuildingSprite, DepositSprite, WorkerSprite
  /ui/panels        BuildPanel, ResearchPanel, InventoryBar, WorkerPanel
  /save             save.ts, migrations.ts
/tests              mirrors /src, plus /fixtures with tiny test data
/tools              dev-only scripts (e.g. export JSON Schema for editor autocomplete)
/docs               design references: transport.md (routes and vehicles by era), properties.md (object properties)
/assets             sprites, tilesets, audio
```

**Boundaries:** `sim`, `map` and `data` never import Phaser, `render` or `ui`. `render` only reads state; `ui` only sends commands. This lets most changes touch one file, its test and its JSON.

---

## 4. Data shapes (draft)

### Object properties (deep simulation)
Objects carry realistic physical, economic and human properties from day one, even before a system uses them, so the simulation can deepen without reworking data. The full list (~110 properties with units, defaults and when each is used) is in `docs/properties.md`. Rules:
- **Units in field names** (`massKg`, `speedKmh`, `meltingPointC`), all metric/SI.
- **Defaults for every optional field**; entries list only what matters.
- **Composition and derived values:** objects list their materials (`composition`); mass, value, fire resistance etc. are computed at load. Writing a value explicitly overrides the derived one.
- **Dormant fields are allowed**; each property is tagged Now / Soon / Later / Future.
- **Range-checked at load**; new properties are always added with a default so existing JSON keeps working.
- **People move at least at horse pace** (`walkKmh` ≥ 10) when not aboard a vehicle; aboard one, the vehicle's speed applies.

### Economy: GORE
- **GORE** (Geosynchronous ORbital Energy) is the currency and the energy unit: **1 GORE = 13,800 kcal = 57.7 MJ**, the energy to lift 1 kg from the surface to geosynchronous orbit.
- **Power is in GORE per tick** (1 GORE/tick ≈ 2.69 hp ≈ 2.0 kW). Energy, fuel, wages and food are all in GORE too, so the game never converts units.
- **Value = mass in kg + power in GORE/tick × value window.** The window starts at **100 ticks** and lives in `/data/economy.json` so it can be tuned after play-testing without touching code.
- Things that run for less than a tick (rockets, engine burns) get a premium equal to the energy they can deliver within the window.
- `valueGore` is what something is **worth**; `cost` is the **materials needed to build it**. They're separate fields.
- Examples: person 74, horse 537, Model T 1,284, Cat 335 excavator 45,054 GORE. Every vehicle's value is in `docs/transport.md`.

### Buildings and tiles
One building per 100 m tile, except housing: a housing tile holds 10 people.

The examples below show core fields only; see `docs/properties.md` for the rest.

### Building
```json
{
  "id": "coal_furnace",
  "name": "Coal Furnace",
  "era": "steam",
  "size": [2, 2],
  "cost": { "brick": 20, "iron_plate": 5 },
  "requiresTech": "basic_metallurgy",
  "workerSlots": [{ "role": "smelter", "count": 1 }],
  "recipes": ["smelt_iron"]
}
```

### Recipe
Recipes are separate from buildings so one building can run many recipes and one recipe can appear in several buildings.
```json
{
  "id": "smelt_iron",
  "inputs": { "iron_ore": 2, "coal": 1 },
  "outputs": { "iron_plate": 1 },
  "durationShifts": 1,
  "requiresEquipment": []
}
```

### Resource
```json
{
  "id": "iron_ore",
  "name": "Iron Ore",
  "category": "mineral",
  "icon": "iron_ore",
  "discoverable": true
}
```

### Technology
```json
{
  "id": "basic_metallurgy",
  "name": "Basic Metallurgy",
  "era": "steam",
  "cost": { "research": 50 },
  "prerequisites": [],
  "unlocks": ["coal_furnace", "smelt_iron"]
}
```

### Worker
```json
{
  "id": "smelter",
  "name": "Smelter",
  "skills": ["smelting"],
  "upkeep": { "food": 1 }
}
```

### Equipment
```json
{
  "id": "steam_drill",
  "name": "Steam Drill",
  "era": "steam",
  "cost": { "iron_plate": 10, "gear": 4 },
  "speedMultiplier": 1.5
}
```

### Era
`order` uses gaps of 100 so new eras can be inserted later (e.g. `250` between Diesel and Jet).
```json
{
  "id": "steam",
  "name": "Steam",
  "order": 100,
  "startYear": 1900
}
```
Current eras: Steam (100, 1900), Diesel (200, 1920), Jet (300, 1945), Information (400, 1970), Modern (500, 2000), Near Future (600, 2030), Orbital (700, 2070), Interplanetary (800, 2120), Interstellar (900, 2200).

### Route
Routes belong to a `network` (road, rail, water, air, space) with a `grade`. A vehicle's speed on a route is the lower of its own speed and the route's `maxSpeedKmh`, so a Model T is slow on a dirt road and fast on gravel. New routes and vehicles slot in without editing existing entries.
```json
{
  "id": "gravel_road",
  "name": "Gravel Road",
  "era": "steam",
  "network": "road",
  "grade": 3,
  "maxSpeedKmh": 40,
  "cost": { "gravel": 5 },
  "buildShiftsPerTile": 1
}
```

### Vehicle
Speeds and capacities are real-world typical values for the era (see `docs/transport.md`).
```json
{
  "id": "freight_wagon",
  "name": "Horse-drawn Freight Wagon",
  "era": "steam",
  "network": "road",
  "minGrade": 2,
  "speedKmh": 4,
  "capacity": { "cargoTonnes": 1.5 },
  "cost": { "wood": 20, "iron_plate": 2 },
  "upkeep": { "fodder": 1 }
}
```

### Terrain
```json
{
  "id": "pine_forest",
  "name": "Pine Forest",
  "buildable": false,
  "moveCost": 2,
  "yields": ["wood"],
  "deposits": { "coal": 0.02 },
  "tileset": "pine_forest"
}
```

### Planet type
The generator layers two noise maps (elevation and moisture); `bands` maps each combination to a terrain. A new kind of planet is mostly a new rule table, not new code.
```json
{
  "id": "temperate",
  "name": "Temperate World",
  "circumferenceKm": 40000,
  "startRegionKm": 40,
  "bands": [
    { "elevation": [0.0, 0.3], "terrain": "water" },
    { "elevation": [0.3, 0.6], "moisture": [0.7, 1.0], "terrain": "wetland" },
    { "elevation": [0.3, 0.6], "moisture": [0.4, 0.7], "terrain": "pine_forest" },
    { "elevation": [0.3, 0.6], "moisture": [0.0, 0.4], "terrain": "plains" }
  ]
}
```

### Fair start
Rules the home planet must satisfy around the landing site. If the generated map falls short, `fairStart.ts` places the missing deposits.
```json
{
  "radiusTiles": 50,
  "requires": [
    { "terrain": "plains", "minTiles": 400 },
    { "resource": "wood", "minTiles": 150 },
    { "resource": "stone", "minDeposits": 2 },
    { "resource": "coal", "minDeposits": 2 },
    { "resource": "iron_ore", "minDeposits": 2 },
    { "resource": "clay", "minDeposits": 1 }
  ]
}
```

---

## 5. Game state (draft)

Each world has its own map, buildings, workers and stockpile. Research is shared.

```ts
interface GameState {
  version: number;                 // for save migrations
  tick: number;
  galaxySeed: number;              // whole galaxy regenerates identically from this
  research: { completed: string[]; active?: { id: string; progress: number } };
  activeWorldId: string;           // which world the player is viewing
  worlds: Record<string, WorldState>; // only explored worlds are stored
  shipments: Shipment[];           // cargo in transit between worlds
}

interface WorldState {
  id: string;
  systemId: string;
  planetType: string;
  seed: number;                    // map regenerates from this; only changes are saved
  stockpile: Record<string, number>;
  buildings: BuildingInstance[];
  workers: WorkerInstance[];
  deposits: Deposit[];             // each has discovered: boolean
  tileChanges: TileChange[];       // edits on top of the generated map
  explored: ExploredGrid;          // 1 km cells, stored compactly (a few MB even for a whole world)
  routes: RouteSegment[];          // roads, rails etc. the player has built
  vehicles: VehicleInstance[];
}

interface Shipment {
  id: string;
  from: string;                    // world id
  to: string;
  cargo: Record<string, number>;
  arrivesAtTick: number;
}

interface WorkerInstance {
  id: string;
  type: string;                    // worker id from workers.json
  homeId?: string;                 // housing building
  targetId?: string;               // building they're assigned to
  shift?: 0 | 1 | 2;               // which of the day's three shifts they work
  state: "idle" | "assigned" | "working";
  x: number; y: number;            // current tile (home or workplace); used by every movement strategy and the renderer
  commuteHours?: number;           // cached commute time from the movement strategy
  path?: [number, number][];       // empty until a path-based strategy exists; here now so no save migration is needed
}
```

---

## 6. Worlds & map generation

### Structure
Galaxy → star systems → planets. Every planet is its own map. The early game happens entirely on the home planet; other worlds unlock as the eras advance.

### Procedural generation
- Everything is generated from seeds: the galaxy from `galaxySeed`, each planet from its own seed. The same seed always produces the same map, so saves store only the seed plus the player's changes.
- A planet is generated the first time it's visited, and only the chunks near the camera are ever generated (see below).
- Noise is our own small seeded function in `noise.ts` (no dependency). It **wraps seamlessly east–west** so a planet's map loops like a globe. This must be right from day one: changing the noise later changes every seed and breaks old saves.

### Map scale & chunks
- **1 tile = 100 m.** The view never shows more than **12 km** (120 tiles) across; the player pans around.
- **The game starts with a county-sized area** (about 40 × 40 km, 400 × 400 tiles) revealed around the landing site. Exploring reveals more, until the explored map can cover a whole planet (an Earth-sized world is about 400,000 tiles around, roughly 51 billion tiles in total).
- **Chunks:** the world is split into 32 × 32-tile chunks (3.2 km square). Chunks near the camera are generated from the seed and cached; chunks far away are discarded and regenerate identically when revisited. Memory use stays flat no matter how much is explored.
- **Exploration** is tracked on a coarse 1 km grid, not per tile, so even a fully explored world takes only a few MB in a save.
- **Zoom levels:** zoomed in, chunks draw full 64×64 tile art. Zoomed out toward 12 km (where a tile is only ~3 px on a phone), each chunk is drawn as one small pre-rendered image instead.
- **World overview:** a separate coarse map of the whole planet for jumping to distant areas.
- **Floating origin:** far from the start, raw coordinates get too large for graphics cards and sprites would jitter. The renderer re-centers its coordinates on the camera; the sim keeps exact integer tile coordinates.
- **The sim doesn't care about map size.** It only processes buildings, workers, routes and vehicles, never empty tiles, so a huge world doesn't slow ticks down. Terrain-aware movement (§7 stage 2) uses distance maps limited to a radius around each building.

### Fair start
The home planet is procedurally generated, but the landing area is guaranteed a reasonable amount of starting resources (see `fairStart.json`). No seed can produce an unwinnable start.

### Stockpiles and shipping
- Each world has its own stockpile. Resources made on one planet can't be used on another until they're shipped there.
- Shipping is a sim system: cargo leaves one stockpile, spends a travel time in transit, then arrives in the destination stockpile.

### Worlds keep working
All explored worlds run every tick, so colonies keep producing while the player looks elsewhere. Only the active world is drawn.

### Terrains
Home planet starts with 8:

| Terrain | Role |
|---|---|
| Plains | default buildable land, farming |
| Pine forest | wood, slows movement |
| Hills | stone, some ores, buildable |
| Snowy mountains | rich minerals, impassable or very slow |
| Red canyon | iron and copper ores, rough ground |
| Desert | sand (glass later), sparse |
| Wetland | clay, slow, needs drainage to build |
| Water | not buildable, fishing and ports later |

Later planets add about 6–8 alien terrains (e.g. ice, volcanic rock, lava, crater dust, crystal fields, toxic marsh), for roughly 14–16 total. Some reuse home-planet tile shapes with a new color palette to keep the art workload down.

### Art
- 64×64 pixel tiles, illustrated fantasy-map style (snowy peaks, dense pines, layered red canyons)
- Each terrain needs a base tile plus edge tiles for blending into neighbors; this is the main cost of adding a terrain
- Resources shown icon-sized with a soft glow
- Undiscovered deposits render as glints until a prospector visits

---

## 7. Workers & movement

### Principle
**Who works where** (scheduling) is separate from **how they get there** (movement). Assignment never changes; the movement strategy can be swapped without touching scheduling, production or saves.

### Worker lifecycle
Every worker is in one state: `idle` (no job), `assigned` (has a job, off shift) or `working` (on shift this tick). Production only counts workers in `working`, so it behaves the same under every movement strategy.

### Commute eats into the shift
A tick is a whole 8-hour shift, so a walk to work is much shorter than a tick. Travel is therefore not a separate state lasting several ticks. Instead, **commute time reduces the worker's output for that shift**:

`output = (8 − commuteHours) ÷ 8`

A worker with a 2-hour commute produces 6/8 of a full shift. Building placement matters from day one, and a very long commute can make a job pointless.

### Movement strategies
All strategies implement one interface in `src/sim/movement/types.ts`:
```ts
interface Movement {
  // hours to travel between two tiles in this world (one way)
  commuteHours(world: WorldState, from: Tile, to: Tile): number;
}
```
Results are cached on the worker (`commuteHours`) and recalculated only when the worker's home or job changes, or the map changes. `config.ts` selects the active strategy. Nothing else changes when it's swapped.

| Stage | Strategy | How commute time is set | Status |
|---|---|---|---|
| 1 | `distance` | Straight-line distance × walking speed. Building placement matters from day one | **Now** |
| 2 | `terrain` | When the map changes, compute a distance map around each building giving true walking time from every tile (mountains, swamps and water cost time). Calculated once, not per worker, so it's cheap on phones | Planned first upgrade |
| 3 | Roads | Workers travel along a road network; dirt road → rail → maglev upgrades follow the eras | Possible later, as an era feature |
| — | Full walking agents | Each worker pathfinds individually | Not planned; only if stages 1–3 fall short |

Walking speed and terrain costs come from JSON (`workers.json`, `terrains.json`), not code.

### Cosmetic walking
Separately from the sim, the renderer can draw small workers walking between home and work purely for looks. It lives entirely in `src/render`, reads `x, y`, `state`, `commuteHours` and the time within the current shift, and can be added or removed at any time without affecting the game.

### Testing
A shared test suite runs against every movement strategy. It checks that commute times are never negative, that a worker living at their workplace has zero commute, that output follows the commute formula, that production only counts working workers, and that saves load the same. Swapping strategies is proven safe by tests, not assumed.

---

## 8. Transport

### Routes and vehicles
The full reviewed list (32 routes, 72 vehicles across 9 eras) lives in `docs/transport.md` and becomes each era's `routes.json` and `vehicles.json`.

- **Real-world speeds and trip times.** Each vehicle uses the typical operating speed and capacity of its real-world counterpart in that era, from a horse-drawn wagon (4 km/h) to a jet airliner (900 km/h) to a 7–9-month chemical-rocket trip to Mars. The speed controls make even long real trips playable (at 1000x a year passes in ~2 s).
- **Speed on a route** = the lower of the vehicle's speed and the route's `maxSpeedKmh`.
- **Older vehicles stay useful.** Nothing is retired when a new era arrives: horses can work alongside maglev trains. Each vehicle keeps its real speed and capacity, so older ones naturally end up doing a small fraction of the total work, but stay handy where they fit (e.g. pack mules on mountain trails with no infrastructure).
- **Expandable:** new routes and vehicles are new JSON entries; existing ones never need editing.

### Within a planet
Goods move physically on the routes and vehicles the player builds (the `logistics` system). Commutes (§7) can later use the same route network (movement stage 3).

### Between worlds
The `shipping` system uses space vehicles from the Orbital era onward; trip times follow the vehicle (e.g. chemical vs. fusion ships).

### FTL starship
The Starship can be faster than light. Its mechanic is **deferred to a future update** and will be designed then. Nothing now should assume interstellar travel is slower than light.

---

## 9. Saving

- Save = `JSON.stringify(GameState)` + data version
- Maps aren't saved tile by tile: each world stores its seed plus `tileChanges`, and the map is regenerated on load
- `migrations.ts` upgrades older saves when the state shape changes
- Storage: localStorage to start; file export/import later

---

## 10. Open questions

- [x] Fresh repo, or a rebuild that replaces steam-to-stars? → fresh repo: breel26/starship
- [x] Phaser 3 or Phaser 4? → Phaser 4, pinned 4.2.1
- [x] Map: one grid or separate maps? → separate map per planet, organized into star systems
- [x] Hand-designed or generated planets? → all procedurally generated from seeds
- [x] Starting conditions? → guaranteed fair start with enough resources to get going
- [x] Shared or per-planet stockpiles? → per planet; resources must be shipped between worlds
- [x] Do other worlds run while the player is away? → yes, every explored world ticks
- [x] Real-time with pause, or fully real-time? → real-time with pause; speeds 1x, 2x, 10x, 100x, 1000x; 1 tick = 8-hour shift = 2 s at 1x (see §2.3 "Time & speed")
- [x] How do workers move? → abstract assignment with distance-based travel now, terrain distance maps next, roads possibly later; cosmetic walking optional (see §7)
- [x] Within one planet: physical transport or a shared stockpile? → physical transport on player-built routes and vehicles (§8, docs/transport.md)
- [x] Map scale? → 100 m tiles, 12 km max view with panning, county-sized start, grows to a whole world via chunks (§6)
- [x] Do older vehicles become obsolete? → no; they stay useful and do a smaller share of the work (§8)
- [ ] Shipping: what carries cargo (ships as units, a route system?) and how travel time is set
- [ ] Galaxy size: how many star systems and planets?
- [ ] Does the game keep progressing while it's closed (offline progress)?
- [x] Era list? → 9 eras: Steam, Diesel, Jet, Information, Modern, Near Future, Orbital, Interplanetary, Interstellar; expandable (§2.1, §4 Era)
- [x] Currency? → GORE; value = mass + 100 ticks of power output (§4 Economy)
- [x] Buildings per tile? → one, except housing (10 people per tile)
- [ ] Should processing ("embodied energy") add value? Iron ore and steel are both 1 GORE/kg under the mass rule
- [ ] Should a fuel's stored energy add to its value?
- [ ] Is housing fixed at 10 people per tile, or can later eras raise it?
- [ ] What defines moving to the next era?
- [ ] FTL Starship mechanic → deferred to a future update (§8)

---

## 11. First milestones

1. Scaffold Vite + TS + Phaser project
2. Zod schemas + loader + registry with cross-reference validation
3. Minimal `GameState`, tick loop, command queue
4. Production system with one building and one recipe, covered by tests
5. Seeded wrapping noise + chunk generation + fair-start check, covered by tests
6. Map scene rendering the home planet and one building
7. Save/load (seed + changes)
