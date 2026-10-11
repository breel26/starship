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
- Scenes: Boot/Preload, Map (one world at a time), StarMap (choose system and planet), UI overlay, Menus
- Reads `GameState` to render; never mutates it directly
- Sends commands to the sim
- An **event bus** pushes sim events to the UI (`research:complete`, `deposit:discovered`, `building:idle`, …)
- Phaser scale mode handles phone and desktop

---

## 3. Repo layout

```
/data
  eras.json
  /steam            resources.json, buildings.json, recipes.json,
                    technologies.json, workers.json, equipment.json
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
                    noise.ts      seeded 2D noise (our own, no dependency)
                    galaxy.ts     generates star systems and planets from a seed
                    generate.ts   builds a planet's tiles from its planet type
                    fairStart.ts  checks/places guaranteed starting resources
                    fog.ts        discovery and visibility
  /render
    /scenes         BootScene, MapScene, StarMapScene, UIScene, MenuScene
    /sprites        BuildingSprite, DepositSprite, WorkerSprite
  /ui/panels        BuildPanel, ResearchPanel, InventoryBar, WorkerPanel
  /save             save.ts, migrations.ts
/tests              mirrors /src, plus /fixtures with tiny test data
/tools              dev-only scripts (e.g. export JSON Schema for editor autocomplete)
/assets             sprites, tilesets, audio
```

**Boundaries:** `sim`, `map` and `data` never import Phaser, `render` or `ui`. `render` only reads state; `ui` only sends commands. This lets most changes touch one file, its test and its JSON.

---

## 4. Data shapes (draft)

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
```json
{
  "id": "steam",
  "name": "Age of Steam",
  "order": 1
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
  "size": [128, 128],
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
  "radius": 12,
  "requires": [
    { "terrain": "plains", "minTiles": 40 },
    { "resource": "wood", "minTiles": 15 },
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
- A planet is generated the first time it's visited.
- Noise is our own small seeded function in `noise.ts` (no dependency).

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

## 8. Saving

- Save = `JSON.stringify(GameState)` + data version
- Maps aren't saved tile by tile: each world stores its seed plus `tileChanges`, and the map is regenerated on load
- `migrations.ts` upgrades older saves when the state shape changes
- Storage: localStorage to start; file export/import later

---

## 9. Open questions

- [x] Fresh repo, or a rebuild that replaces steam-to-stars? → fresh repo: breel26/starship
- [x] Phaser 3 or Phaser 4? → Phaser 4, pinned 4.2.1
- [x] Map: one grid or separate maps? → separate map per planet, organized into star systems
- [x] Hand-designed or generated planets? → all procedurally generated from seeds
- [x] Starting conditions? → guaranteed fair start with enough resources to get going
- [x] Shared or per-planet stockpiles? → per planet; resources must be shipped between worlds
- [x] Do other worlds run while the player is away? → yes, every explored world ticks
- [x] Real-time with pause, or fully real-time? → real-time with pause; speeds 1x, 2x, 10x, 100x, 1000x; 1 tick = 8-hour shift = 2 s at 1x (see §2.3 "Time & speed")
- [x] How do workers move? → abstract assignment with distance-based travel now, terrain distance maps next, roads possibly later; cosmetic walking optional (see §7)
- [ ] Within one planet: physical transport (carts, rail) or one shared stockpile per planet?
- [ ] Shipping: what carries cargo (ships as units, a route system?) and how travel time is set
- [ ] Galaxy size: how many star systems and planets?
- [ ] Does the game keep progressing while it's closed (offline progress)?
- [ ] Era list and what defines moving to the next era

---

## 10. First milestones

1. Scaffold Vite + TS + Phaser project
2. Zod schemas + loader + registry with cross-reference validation
3. Minimal `GameState`, tick loop, command queue
4. Production system with one building and one recipe, covered by tests
5. Seeded noise + home planet generation with fair-start check, covered by tests
6. Map scene rendering the home planet and one building
7. Save/load (seed + changes)
