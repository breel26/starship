# Starship — Working Rules

Read this before making any change. See ARCHITECTURE.md for the design.

## Ground rules

1. **Talk first.** If a change touches more than ~3 files, adds a new system, adds a dependency, or changes the architecture, propose a plan and wait for Buck's OK before writing code.
2. **Test everything.** Every sim or data change comes with tests. `npm test` and the type-check must pass before committing. Report the results.
3. **Minimal dependencies.**
   - Runtime: Phaser and Zod 4 (Mini only). Nothing else.
   - Dev: TypeScript, Vite, Vitest.
   - Any new package needs Buck's approval and a stated reason. Pin exact versions in `package.json`.
4. **Small files.** One system, scene, sprite type or UI panel per file. Aim for under ~300 lines; split a file that grows past that.
5. **Buck play-tests before `main`.** Changes go to `main` only after Buck has played them and said they look good.

## Boundaries

- `src/sim`, `src/map` and `src/data` never import Phaser, `src/render` or `src/ui`.
- `src/render` reads game state; it never mutates it.
- `src/ui` changes the game only by sending commands to the sim.
- All game content lives in `/data` JSON. No content (names, costs, recipes, numbers) hard-coded in `src`.

## Zod 4 Mini idioms

Use `zod/mini` in game code, never the classic `zod` import and never Zod 3 syntax. Mini uses functions, not chained methods. These patterns are verified against zod 4.6:

```ts
import * as z from "zod/mini";
import en from "zod/v4/locales/en.js";
z.config(en()); // once at startup; without it errors just say "Invalid input"

const Id = z.string().check(z.minLength(1), z.regex(/^[a-z0-9_]+$/));
const Amounts = z.record(z.string(), z.number().check(z.positive())); // record needs key AND value schema
const Slot = z.object({ role: Id, count: z.int().check(z.gte(1)) });

export const Building = z.strictObject({        // strictObject rejects unknown keys
  id: Id,
  size: z.tuple([z.int(), z.int()]),
  cost: Amounts,
  requiresTech: z.optional(Id),                 // NOT Id.optional()
  workerSlots: z._default(z.array(Slot), []),   // NOT .default([])
  category: z.enum(["production", "storage"]),
});
export type Building = z.infer<typeof Building>;

const r = Building.safeParse(json);
if (!r.success) throw new Error(z.prettifyError(r.error)); // NOT .format() / .flatten()

z.extend(Building, { tier: z.int() });  // NOT Building.extend(...)
z.partial(Building);                    // NOT Building.partial()
```

Checks go inside `.check(...)`: `z.minLength`, `z.maxLength`, `z.gte`, `z.lte`, `z.positive`, `z.regex`.
JSON Schema export (`toJSONSchema` from `"zod"`) is for dev scripts only, never game code.
