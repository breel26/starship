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
| research | Turns lab and university output into research points and applies them to the player's chosen technologies; unlocks content (research is global, not per world). See §10 |
| discovery | Rolls for chance discoveries from labs and universities, including discoveries outside a lab's own field. See §10 |
| scientists | Brings great scientists into the game, applies their breakthroughs and pool bonuses. See §10 |
| construction | Runs build and manufacturing activities: consumes materials, applies assigned labor and equipment, records progress. See §11 |
| schedule | Critical path, leveling and previews for all activities; runs the active planner. See §11 |
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
  balance.json      every tuning knob and global multiplier (see §12)
  economy.json      reference constants (GORE in joules, hp in watts)
  fields.json       research fields (physics, chemistry, medicine; more later)
  /steam            resources.json, buildings.json, recipes.json,
                    technologies.json, scientists.json, workers.json, equipment.json,
                    routes.json, vehicles.json
  /…                one folder per era, same files
  /world            terrains.json, planetTypes.json, galaxy.json, fairStart.json
/src
  main.ts           starts Phaser
  config.ts         engine-only settings (art tile pixels, chunk size); gameplay numbers live in /data
  /core             shared types, ids, event bus, rng (no game logic)
  /data
    /schemas        one Zod schema file per content type
    loader.ts       loads + merges era folders and packs
    crossref.ts     checks every id reference resolves
    registry.ts     read-only lookup by id
  /sim
    state.ts  tick.ts  commands.ts
    /systems        production.ts, scheduling.ts, research.ts,
                    prospecting.ts, logistics.ts, shipping.ts, movement.ts,
                    construction.ts, discovery.ts, scientists.ts
    /schedule       cpm.ts, leveling.ts, preview.ts (see §11)
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
  /planner          Planner interface + builtin planner (AI planners plug in here later); no Phaser
  /render
    /scenes         BootScene, MapScene, WorldOverviewScene, StarMapScene,
                    UIScene, MenuScene
    chunkView.ts    loads/unloads chunk layers near the camera; full art zoomed in,
                    one pre-rendered image per chunk zoomed out
    origin.ts       re-centers render coordinates on the camera (no far-away jitter)
    /sprites        BuildingSprite, DepositSprite, WorkerSprite
  /ui/panels        BuildPanel, ResearchPanel, InventoryBar, WorkerPanel
  /ui/schedule      GanttView, ActivitySheet (tabs), SchedulePreview, TimeCostCurve, ResourceProfile
  /save             save.ts, migrations.ts
/tests              mirrors /src, plus /fixtures with tiny test data
/tools              dev-only scripts: JSON Schema export for editor autocomplete, balanceReport.ts (§12)
/docs               design references: transport.md (routes and vehicles by era), properties.md (object properties),
                    tech-tree-physics.md (example research tree), open-questions.md (unresolved design questions)
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
- **Value = mass + manufacturing energy + stored energy + work premium.** Mass is 1 GORE per kg. Manufacturing energy is the energy spent making it, summed through the recipe chain (raw materials have none). Stored energy is what fuel, food and (later) batteries hold: 1 kg of coal is worth about 1.42 GORE before mining costs, 1 kg of diesel about 1.78 plus refining. The work premium is power in GORE/tick × the value window, for things that can do work. The window starts at **100 ticks** and lives in `/data/balance.json` so it can be tuned after play-testing without touching code.
- **Manufacturing energy is calculated, not typed in:** each recipe run uses (building power + workers' work output) × duration; outputs are worth their inputs' value plus that energy. The registry computes this at load by walking the recipe graph, and an item can override it.
- Things that run for less than a tick (rockets, engine burns) get a premium equal to the energy they can deliver within the window.
- `valueGore` is what something is **worth**; `cost` is the **materials needed to build it**. They're separate fields.
- Examples: person 74, horse 537, Model T 1,284, Cat 335 excavator 45,054 GORE. Every vehicle's value is in `docs/transport.md`.

### Buildings and tiles
One building per 100 m tile, except housing. Housing density is set per housing type: early housing holds 10 people per tile, and later types (apartments, arcologies) hold more.

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
See §10 for how research works.
```json
{
  "id": "basic_metallurgy",
  "name": "Basic Metallurgy",
  "era": "steam",
  "year": 1900,
  "field": "chemistry",
  "costRp": 500,
  "prerequisites": [],
  "unlocks": ["coal_furnace", "smelt_iron"]
}
```

### Research field
Fields are data, so new ones (e.g. engineering, computing, biology) can be added in later eras.
```json
{ "id": "physics", "name": "Physics", "era": "steam" }
```

### Lab (a building with a `research` block)
```json
{
  "id": "physics_lab",
  "name": "Physics Laboratory",
  "era": "steam",
  "research": {
    "field": "physics",
    "rpPerTick": 2,
    "fundingGorePerTick": 5,
    "discoveryChancePerTick": 0.002,
    "discoveryFields": { "physics": 0.7, "chemistry": 0.2, "medicine": 0.1 }
  },
  "workerSlots": [{ "role": "scientist", "count": 4 }]
}
```

### Great scientist
```json
{
  "id": "marie_curie",
  "name": "Marie Curie",
  "era": "steam",
  "arrival": { "earliestYear": 1900, "latestYear": 1910 },
  "breakthrough": "radioactivity",
  "careerBonus": { "fields": ["physics", "chemistry"], "percent": 15, "careerTicks": 3285 }
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
  research: ResearchState;         // see below
  activeWorldId: string;           // which world the player is viewing
  worlds: Record<string, WorldState>; // only explored worlds are stored
  shipments: Shipment[];           // cargo in transit between worlds
}

interface ResearchState {
  completed: string[];            // unlocked techs; "discovered" is derived from prerequisites
  progress: Record<string, number>;      // tech id → research points so far
  queues: Record<string, string[]>;      // field id → techs the player queued
  generalPool: number;                   // university points not yet assigned
  scientists: ActiveScientist[];         // great scientists currently working
  scientistsArrived: string[];           // ids, so each appears once per game
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
- **Style: top-down pixel art, mixed resolution** (chosen after comparing 16-bit, detailed 32 px, isometric, 64 px and mixed mockups):
  - **Terrain** is drawn as **32×32 px tiles shown at 2×**, so every map tile is 64 px on screen (1 tile = 100 m).
  - **Buildings, vehicles and other things the player makes** are drawn at **native 64×64 px**, so they stand out from the land and can show their state (smoke, scaffolding, staked foundations, working machines).
  - Trees, hills and mountains belong to the terrain layer (32 px) and overlap the tile above for a 3/4 look.
  - Rendered with nearest-neighbor scaling (no smoothing). In Phaser: `pixelArt: true`.
- Illustrated fantasy-map feel: snowy peaks, dense pines, layered red canyons, shorelines with foam
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
Results are cached on the worker (`commuteHours`) and recalculated only when the worker's home or job changes, or the map changes. `balance.json` selects the active strategy. Nothing else changes when it's swapped.

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

## 9. Gameplay overview

### Goal
Discover resources, turn them into buildings and manufactured items, and combine those into ever more complex assemblies until you can build a **starship** that leaves the planet and explores the galaxy. Early starships use known technology; later, research can discover **FTL drives**, with several possible versions (FTL mechanic deferred, see §8).

### Core loop
1. **Explore** to find resources (prospectors reveal deposits).
2. **Build infrastructure** to extract them and move them to manufacturing plants (routes and vehicles, §8).
3. **Manufacture** simple items, then combine them into more complex assemblies (machinery, computers, mills, power plants, components…).
4. **Research** new manufacturing processes and items (§10).
5. Repeat at larger scale, era by era, toward the starship.

### Scheduling construction and manufacturing
The second big mechanic is scheduling builds with limited resources. Everything has a **time** and a **cost**.
- **Every job** (a building, a vehicle, a batch of items) automatically gets a default allocation of **materials, labor and equipment** from its data.
- **The player can adjust the allocation** to speed a job up or slow it down: more workers and equipment finish sooner, fewer stretch it out and free resources for other jobs. Materials are always needed in full.
- **Every buildable item has its own build screen** showing its materials, labor, equipment, time, cost in GORE and the effect of the player's adjustments.
- Full design: §11 (P6-style scheduling, resource assignments and the planner).

---

## 10. Research & discovery

### Research points and fields
- Research produces **research points (RP)** in **fields**: **physics, chemistry and medicine** to start. Fields are data (`fields.json`), so later eras can add more (e.g. engineering, computing, biology).
- Every technology belongs to one field and has a cost in RP of that field.
- The player sets a **research queue per field**. Each tick, a field's RP flow into the first tech in its queue; leftovers carry over to the next.

### Labs
- **Physics, chemistry and medical labs** are buildings. Each produces RP in its own field.
- **Labs cost GORE to run:** each lab has a funding level in GORE per tick. Output scales with funding and with its staffed scientists, so research competes with construction and manufacturing for the same GORE.
- Proposed output rule: `RP per tick = rpPerTick × (staffed scientists ÷ slots) × (funding ÷ fundingGorePerTick)`, with funding capped at 100% for now. Player-set underfunding slows a lab rather than stopping it.

### Universities
- Universities add RP to a **general pool** that can be applied to any field's queue (the player assigns it, or it splits automatically across active queues).
- They also have a chance of making discoveries (below) and, later, train scientists (people `education`, properties P16).

### Chance discoveries (including across fields)
- Every lab and university has a small **discovery chance each tick**. When it hits, the discovery lands in a field picked by that building's `discoveryFields` weights, so **a physics lab can make a chemistry discovery** and **a medical lab can make a chemistry discovery**.
- A discovery adds a **lump of RP to a discovered tech** in that field (enough to unlock a cheap one outright).
- Chances and weights are data per building type, so they can be tuned in play-testing.
- All randomness uses the seeded generator in `src/core/rng.ts`, so a save replays identically and tests can check results exactly.

### Great scientists
- Great scientists are **real historical figures** (Curie, Einstein, Bohr, Meitner…), listed in each era's `scientists.json`.
- Each arrives at a random point inside their historical year window, **once per game**, provided their breakthrough tech's prerequisites are done; if not, they arrive as soon as they are. Arrival is more likely the more the player invests in research.
- **On arrival their breakthrough tech unlocks immediately**, no RP needed. They then work for a **career**, adding a **percentage bonus** to the RP of one or more fields.
- The UI announces arrivals and breakthroughs, with an auto-pause option (§2.3).

### Tech states and the tree
- **Eras don't gate research.** A tech's `era` and `year` are labels for flavor and pacing only.
- Each tech is in one of three states:
  - **Hidden:** some prerequisites aren't done; not shown.
  - **Discovered:** every prerequisite is done. It appears on the tree with its **RP cost** and **estimated time to unlock**, and can be queued.
  - **Unlocked:** its RP are fully paid (or a great scientist unlocked it); its content becomes available.
- **Estimated time = RP remaining ÷ RP per tick going into it.** Adding RP (more labs, more funding, university points, scientist bonuses) shortens it, and the estimate updates live, shown in ticks, game days and real time at the current speed.
- FTL drive variants will be techs deep in the tree, reachable once their prerequisites are done.
- A worked example tree is in `docs/tech-tree-physics.md`.

### What research unlocks
Technologies unlock content by id: recipes (new manufacturing processes), buildings, items, equipment, routes, vehicles, housing types, labs, and new research fields. Anything in the data can be gated behind a tech.

### Testing
Research math, discovery rolls (with a fixed seed), scientist arrival windows, queue overflow and era reveal rules all get unit tests in `tests/sim/systems/`.

---

## 11. Scheduling (P6-style) & the planner

Construction and manufacturing are scheduled like **Primavera P6**: activities with predecessors and successors, resource assignments, a critical path and a Schedule button that shows what a change does to the timeline and GORE. A built-in **planner** does the scheduling by default; the player can change anything at any time.

### Projects and activities
- Every construction or manufacturing job is an **activity**. Activities belong to **projects** (P6's WBS): "Build Iron Works" contains survey, foundation, furnace and so on. Eventually the Starship is one very large project.
- **Relationships:** Finish-to-Start, Start-to-Start, Finish-to-Finish and Start-to-Finish, each with an optional lag in ticks. The engine supports all four from the start; the UI defaults to Finish-to-Start.
- **Automatic links:** when an activity needs another activity's output (steel for a furnace) or a material delivery, the link is created automatically. Deliveries by route and vehicle are activities too.
- **Constraints** (as in P6): start on or after, finish on or before, and priority.

### Resources drive the duration (P6 "Fixed Units")
Each buildable item's data defines its **work content**, not its duration:
```json
"work": {
  "labor":     { "laborer": 120, "mason": 20 },
  "equipment": { },
  "materials": { "brick": 20000, "iron_plate": 5000 },
  "crew":      { "laborer": { "default": 4, "max": 8 }, "mason": { "default": 1, "max": 2 } },
  "overheadGorePerTick": 2
}
```
- Labor and equipment are in worker-ticks and machine-ticks; materials (kg) are always needed in full.
- **Duration = the longest of (work units ÷ assigned units) across its resources.** The slowest resource sets the pace. Past a crew's `max`, extra units add little (crowding; the curve is a knob in `balance.json`).
- Example, Coal Furnace: default crew → 30 ticks; 8 laborers + 1 mason → 20 ticks (the mason becomes the bottleneck); 8 laborers + 2 masons → 15 ticks.

### Resources, GORE and time
Every resource has a GORE rate, so time and cost are always connected and shown together:
| Resource | GORE cost |
|---|---|
| Labor | the worker's wage per tick × ticks assigned |
| Equipment | fuel + wear per tick × ticks assigned |
| Materials | their GORE value |
| Overhead | the activity's site cost per tick while it's open |

Finishing sooner cuts overhead; overloading a crew past its `max` wastes labor. The player can see that trade-off directly (below).

### The schedule engine
Pure TypeScript in `src/sim/schedule/` (no Phaser), heavily unit-tested:
- `cpm.ts`: forward and backward passes, early/late dates, total float, critical path.
- `leveling.ts`: delays non-critical work within its float when workers or equipment are over-allocated.
- `preview.ts`: runs a change on a copy of the schedule and returns the before/after difference (finish dates, GORE, critical path, resource conflicts) without committing anything.
- Progress, actual start/finish and slips from shortages or breakdowns feed back in every tick.

### The planner (built-in now, pluggable later)
- **The player sets goals; the planner plans.** The player chooses what to build ("Iron Works", "200 t of steel"); the planner creates and links activities, sizes crews, assigns equipment and levels resources across all projects, and replans as conditions change.
- **Policy dial** per project or globally: **Fastest**, **Balanced** or **Cheapest** (how it trades time for GORE).
- **Modes** per project:
  - **Auto:** the planner applies its own changes.
  - **Advise:** it proposes; the player reviews the before/after preview and accepts or rejects.
  - **Manual:** it only runs the critical-path math.
- **Player changes are pinned.** Any crew, link, constraint or priority the player sets is kept; the planner plans around it. Unpinning hands it back.
- **It explains itself:** each decision carries a short note, e.g. "Delayed 6 ticks: both masons are on the Coal Furnace (critical)."
- **Planning skill is progression:** the planner starts crude in 1900 (first come, first served, no leveling) and improves as the player researches techs like Gantt Charts (1910s), Critical Path Method (1957) and Computers, or hires a skilled project manager. Which capabilities each tech unlocks is data.

### Plugging in a real AI agent later
The planner sits behind one interface, so a real AI agent such as Claude can replace or advise it without changing the sim:
```ts
interface Planner {
  id: string;                                        // "builtin", later e.g. "claude"
  plan(input: PlannerInput): Promise<PlannerProposal>;
}
interface PlannerInput {                             // plain JSON, easy to send to an AI model
  snapshot: ScheduleSnapshot;                        // projects, activities, resources, stockpiles, tick
  goals: Goal[];
  pins: Pin[];                                       // player decisions the planner must keep
  policy: "fastest" | "balanced" | "cheapest";
}
interface PlannerProposal {
  commands: Command[];                               // the same commands the player's UI sends
  notes: { activityId?: string; text: string }[];    // explanations shown to the player
}
```
- **Proposals are only commands.** The planner never edits game state directly; its commands go through the same validation as the player's, so any planner, built-in or AI, can't break rules or cheat.
- **Asynchronous by design:** `plan()` returns a Promise, so a planner that calls a remote service fits without changes. If it is slow or fails, the game falls back to the built-in planner.
- **Saves stay reproducible:** saves record the commands that were applied, not the planner's reasoning, so a game replays identically whichever planner produced them.
- **Schemas double as tool definitions:** the Zod schemas for commands and snapshots can be exported as JSON Schema and given to an AI model as its tool definitions.
- An AI planner would need a network connection, an API key and a new dependency (or plain `fetch`); per the ground rules, that's a decision for later, with Buck's approval.

### Screens (phone-first, fuller layout on desktop)
1. **Gantt view:** activity list with bars, critical path in red, float, relationship lines; the timeline scrolls sideways.
2. **Activity details** in a bottom sheet with tabs:
   - **General:** dates, float, % complete, constraints, pinned or planner-managed
   - **Resources:** each resource's units per tick, total units, GORE/tick and GORE total; add or remove resources here
   - **Predecessors** and **Successors**
   - **Costs:** GORE split into labor, equipment, materials and overhead
   - **Planner notes**
3. **Schedule button:** recalculates and shows the before/after preview (finish dates, total GORE, critical path, conflicts) with Apply or Cancel.
4. **Time–cost curve** on the Resources tab: crew size against finish date and GORE.
5. **Resource usage profile:** demand vs available workers and equipment per tick.
6. **Baselines:** save one and compare progress against it.

### State (sketch)
```ts
interface ScheduleState {
  projects: Project[];             // goal, mode (auto/advise/manual), policy
  activities: Activity[];          // work, assignments, links, constraints, pins, status, progress, actual dates
  baselines: Baseline[];
  plannerId: string;               // "builtin" for now
}
```

---

## 12. Balancing & tuning

The game will be rebalanced continuously: RP earned and RP costs, durations, GORE costs and values, resource availability and more. **Every number that affects gameplay is adjustable in JSON**; none is hard-coded in `src`.

### Two layers of knobs
1. **Per-item values** in the content files: each tech's `costRp`, each recipe's `durationShifts`, each lab's `rpPerTick` and `fundingGorePerTick`, each deposit's size, each vehicle's speed, and so on. Use these to fix one thing.
2. **Global multipliers** in **`/data/balance.json`**: one place to scale a whole category at once (e.g. "all research costs ×1.2") without editing hundreds of entries. Multipliers can also be set per research field, per era or per resource where it's useful.

The registry applies the multipliers once at load, so the sim always reads final numbers, and the build screens and tech tree show the balanced values.

### What `balance.json` covers (starting values)
```json
{
  "time":      { "secondsPerTickAt1x": 2, "speeds": [1, 2, 10, 100, 1000] },
  "economy":   { "valueWindowTicks": 100, "goreCostMultiplier": 1.0 },
  "research":  {
    "rpEarnedMultiplier": 1.0,
    "rpCostMultiplier": 1.0,
    "byField": { "physics": { "rpCostMultiplier": 1.0 } },
    "discoveryChanceMultiplier": 1.0,
    "discoveryRpLump": 0.25,
    "scientistBonusMultiplier": 1.0,
    "scientistArrivalMultiplier": 1.0
  },
  "construction": {
    "durationMultiplier": 1.0,
    "materialCostMultiplier": 1.0,
    "laborSpeedupLimit": 2.0
  },
  "production": { "durationMultiplier": 1.0, "outputMultiplier": 1.0 },
  "resources": {
    "abundanceMultiplier": 1.0,
    "depositSizeMultiplier": 1.0,
    "byResource": { "iron_ore": { "abundanceMultiplier": 1.0 } }
  },
  "workers":   { "commuteMultiplier": 1.0, "foodNeedMultiplier": 1.0, "wageMultiplier": 1.0 },
  "transport": { "speedMultiplier": 1.0, "capacityMultiplier": 1.0 },
  "world":     { "tileMeters": 100, "startRegionKm": 40, "fairStartMultiplier": 1.0 }
}
```
New knobs are added with a default of 1.0 (or the current value), so existing saves and data keep working.

### What stays in code
Only things that aren't gameplay choices: unit definitions (1 GORE = 57,739,200 J, 1 hp = 745.7 W), engine settings (art tile size in pixels, chunk size, render budgets) and the rules themselves (formulas). The physical constants live in `/data/economy.json` as reference values; `balance.json` holds the knobs.

### Tools for rebalancing
- **Live reload while developing:** Vite reloads JSON on save, so a changed number shows up in the running game within a second.
- **Balance report** (`tools/balanceReport.ts`, dev-only): prints the numbers that matter for pacing, e.g. total RP and time for each tech tree at a given research rate, item values in GORE, production rates per building, and anything far outside its neighbors.
- **Pacing tests:** because the sim is plain TypeScript, a test can run a scripted game for decades at full speed (no rendering) and check milestones, e.g. "with 2 labs, Nuclear Fission lands between 1930 and 1950". These start as warnings and tighten as the game settles.
- **Presets later:** a different balance file (e.g. `balance.easy.json`) can be swapped in for difficulty levels without touching content.
- **Saves record which balance version they were played with**, so play-test feedback can be traced to the numbers in use.

---

## 13. Saving

- Save = `JSON.stringify(GameState)` + data version
- Maps aren't saved tile by tile: each world stores its seed plus `tileChanges`, and the map is regenerated on load
- `migrations.ts` upgrades older saves when the state shape changes
- Storage: localStorage to start; file export/import later

---

## 14. Open questions

The full, categorized list of unresolved questions is in `docs/open-questions.md`. The checklist below records decisions as they're made.

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
- [x] Does manufacturing add value? → yes; value includes the energy spent making it (embodied energy)
- [x] Does stored energy add value? → yes; fuel, food and batteries are worth their mass plus the energy they hold
- [x] Housing density? → starts at 10 people per tile and rises with later housing types
- [x] Do eras gate research? → no; a tech is discovered as soon as its prerequisites are done (§10)
- [ ] Is there a minimum research time per tech? (docs/tech-tree-physics.md)
- [ ] What, if anything, does "the current era" mean for gameplay now that it doesn't gate research?
- [x] Scheduling? → P6-style activities, relationships, resource assignments and Schedule preview; built-in planner runs it, player can override anytime (§11)
- [x] AI agent? → built-in planner now; Planner interface lets a real AI agent (e.g. Claude) plug in later (§11)
- [ ] Scheduling UI: confirm phone-first layout; relationship types in the UI beyond Finish-to-Start; mockups of Gantt and Resources screens
- [ ] Crowding curve, overhead rates and partial material delivery (§11, tunable in balance.json)
- [ ] Research tuning: lab output rule, discovery chances, scientist bonuses (§10); all adjustable in balance.json (§12)
- [x] Great scientists? → real historical figures; their breakthrough unlocks immediately on arrival
- [ ] FTL Starship mechanic → deferred to a future update (§8)

---

## 15. First milestones

1. Scaffold Vite + TS + Phaser project
2. Zod schemas + loader + registry with cross-reference validation
3. Minimal `GameState`, tick loop, command queue
4. Production system with one building and one recipe, covered by tests
5. Seeded wrapping noise + chunk generation + fair-start check, covered by tests
6. Map scene rendering the home planet and one building
7. Save/load (seed + changes)
