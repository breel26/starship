# Starship — Architecture

A resource / exploration / crafting / scheduling game that starts with 1900-era technology and builds toward a starship and travel to other star systems.

**Stack:** TypeScript · Phaser (version TBD, see §8) · Zod 4 Mini · Vite · Vitest
**Principle:** data-driven design. All game content (resources, technologies, buildings, workers, equipment, recipes, eras) lives in JSON. Code defines *rules*; JSON defines *content*.
**Working rules:** see CLAUDE.md.

### Dependencies
| Package | Where | Why |
|---|---|---|
| Phaser | runtime | rendering, input, scenes |
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
Every piece of game content is a JSON file under `/data`, split into one folder per era so files stay small. Nothing about content is hard-coded.

Expansion and mod packs use the same folder format and go through the same validation, so the game is expandable without code changes.

### 2.2 Registry
At boot:
1. Load all JSON files (built-in, then any packs) and merge them
2. Validate each against a Zod 4 Mini schema
3. Check cross-references (e.g. a recipe input `iron_ore` must exist in `resources.json`; a building's `requiresTech` must exist in `technologies.json`)
4. Index everything by `id` into a read-only registry

Bad data fails loudly at startup, never mid-game.

### 2.3 Simulation
- Pure TS systems run on a **fixed tick** (starting point: 4 ticks/sec)
- World state is a single plain object (`GameState`)
- Systems read the registry + state and produce the next state
- Player actions arrive as **commands** (e.g. `{ type: "build", buildingId: "coal_furnace", x: 12, y: 8 }`), validated and applied at the start of a tick

Initial systems:
| System | Responsibility |
|---|---|
| production | Runs recipes in buildings, consumes inputs, produces outputs |
| scheduling | Assigns workers to jobs and building slots |
| research | Progresses technologies, unlocks content |
| prospecting | Reveals mineral deposits when a prospector visits an area |
| logistics | Moves resources between storage and buildings |

### 2.4 Presentation (Phaser)
- Scenes: Boot/Preload, Map, UI overlay, Menus
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
                    prospecting.ts, logistics.ts
  /map              generate.ts, fog.ts (pure logic, no Phaser)
  /render
    /scenes         BootScene, MapScene, UIScene, MenuScene
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
  "durationTicks": 40,
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

---

## 5. Game state (draft)

```ts
interface GameState {
  version: number;          // for save migrations
  tick: number;
  inventory: Record<string, number>;
  buildings: BuildingInstance[];
  workers: WorkerInstance[];
  research: { completed: string[]; active?: { id: string; progress: number } };
  map: {
    width: number;
    height: number;
    tiles: Tile[];
    deposits: Deposit[];    // each has discovered: boolean
  };
}
```

---

## 6. Art & map

- 64×64 pixel tiles, illustrated fantasy-map style (snowy peaks, dense pines, layered red canyons)
- Resources shown icon-sized with a soft glow
- Undiscovered deposits render as glints until a prospector visits

---

## 7. Saving

- Save = `JSON.stringify(GameState)` + data version
- `migrations.ts` upgrades older saves when the state shape changes
- Storage: localStorage to start; file export/import later

---

## 8. Open questions

- [x] Fresh repo, or a rebuild that replaces steam-to-stars? → fresh repo: breel26/starship
- [ ] Phaser 3 or Phaser 4 (current release is 4.2)?
- [ ] Map: one tile grid that expands over eras, or separate maps per planet/system?
- [ ] Real-time with pause, or fully real-time?
- [ ] How do workers move — pathfinding on the map, or abstract assignment?
- [ ] Does logistics need physical transport (carts, rail) or is storage global?
- [ ] Era list and what defines moving to the next era

---

## 9. First milestones

1. Scaffold Vite + TS + Phaser project
2. Zod schemas + loader + registry with cross-reference validation
3. Minimal `GameState`, tick loop, command queue
4. Production system with one building and one recipe, covered by tests
5. Map scene rendering tiles and one building
6. Save/load
