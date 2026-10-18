# Starship — Progress

Where the project stands, what's been done and what's next. Update this file at the end of each work session.

**Last updated:** 18 Oct 2026
**Phase:** Planning and design. No game code yet.
**Next step:** Answer the open questions that block coding, then scaffold milestone 1.

---

## Status at a glance

| Area | Status | Where |
|---|---|---|
| Working rules | ✅ Done | CLAUDE.md |
| Architecture | ✅ Drafted, still growing | ARCHITECTURE.md |
| Transport list (32 routes, 72 vehicles, 9 eras) | ✅ Reviewed | docs/transport.md |
| Object properties (~110 properties) | ✅ Reviewed | docs/properties.md |
| Economy (GORE) | ✅ Decided | ARCHITECTURE.md §4, docs/properties.md |
| Research system | ✅ Designed; example physics tree drafted | ARCHITECTURE.md §10, docs/tech-tree-physics.md |
| Scheduling (P6-style) and planner | ✅ Designed | ARCHITECTURE.md §11 |
| Balancing approach | ✅ Designed | ARCHITECTURE.md §12 |
| Art style | ✅ Chosen | ARCHITECTURE.md §6 "Art" |
| Home screen mockup (desktop and phone) | ✅ Done in final art style | Design canvas "Starship Home Screen" (private Claude artifact) |
| Schedule (Gantt) screen mockup | ⬜ Not started | — |
| Other screen mockups | ⬜ Not started | — |
| Chemistry and medicine tech trees | ⬜ Not started | — |
| Open questions | 🟡 26 open | docs/open-questions.md |
| Game code | ⬜ Not started | — |

---

## Key decisions so far

**Tech and process**
- TypeScript + **Phaser 4** (pinned 4.2.1) + **Zod 4 Mini**; dev tools Vite and Vitest. Nothing else without approval.
- Data-driven: all content in JSON under `/data`; **every gameplay number is tunable** (global knobs in `balance.json`).
- Simulation is plain TypeScript with no Phaser; Phaser only draws and sends commands.
- Ground rules: talk before big changes, test everything, small files, Buck play-tests before `main`.

**World**
- Separate procedurally generated map per planet, grouped into star systems; generated from seeds, saves store only changes.
- **100 m tiles**, view up to 12 km with panning; start county-sized (~40 km), grows to a whole world via chunks.
- Guaranteed fair start; per-planet stockpiles that must be shipped between worlds; every explored world keeps running.

**Time**
- One tick = one 8-hour shift = **2 s at 1×**, so a day is 6 s. Pause, 1×, 2×, 10×, 100×, 1000×.

**People and transport**
- Workers are assigned abstractly; commute time reduces a shift's output; movement is swappable (distance now, terrain later, roads maybe).
- People move at least at horse pace (10 km/h) when not on a vehicle.
- Real-world speeds for all routes and vehicles; older vehicles stay useful.

**Economy**
- Currency **GORE** = energy to lift 1 kg to geosynchronous orbit (13,800 kcal, 57.7 MJ).
- Value = mass + manufacturing energy + stored energy + work premium (power in GORE/tick × 100-tick window).
- One building per tile, except housing (10 people per tile to start, rising with later housing types).

**Research**
- RP in physics, chemistry, medicine (more fields later); labs cost GORE to run; universities add a shared pool.
- A tech becomes available as soon as its prerequisites are done (eras don't gate research); time estimates update live.
- Chance discoveries can land in other fields; real historical great scientists unlock their discovery on arrival.

**Scheduling**
- Primavera P6-style activities, relationships, resource assignments and critical path; resources drive duration.
- A built-in planner schedules automatically; the player can override anything (Auto / Advise / Manual).
- Planner sits behind an interface so a real AI agent (e.g. Claude) can plug in later.

**Art**
- Top-down pixel art: 32 px terrain shown at 2×, 64 px buildings and vehicles.

---

## Log

| Date | Work |
|---|---|
| 11 Oct 2026 | Project started as a fresh repo (breel26/starship). Architecture, working rules, Phaser 4 and Zod 4 Mini choices. World, time, workers and transport design. Transport and properties lists drafted and reviewed. GORE economy. Research and discovery system with example physics tree. Balancing approach. P6-style scheduling with pluggable planner. |
| 12 Oct 2026 | Home screen mockup (desktop and phone): map, screen tabs, build dropdowns, speed controls, selection panel. |
| 17 Oct 2026 | Compared five pixel-art map styles; chose mixed 32/64 px. Home screen redone in that style with a factory and steam shovel. Art decision recorded. |
| 18 Oct 2026 | Collected all open design questions into docs/open-questions.md. Added this progress file. |

---

## Next steps

1. **Answer blocking questions** in docs/open-questions.md, especially Q26 (when to start coding) and the research and scheduling tuning questions.
2. **Mock up the Schedule screen** (Gantt chart, activity panel, Resources tab, before/after preview).
3. **Milestone 1:** scaffold Vite + TypeScript + Phaser 4 (ARCHITECTURE.md §15).
4. **Milestone 2:** Zod schemas, loader and registry with cross-reference validation, plus `balance.json`.
5. **Milestone 3+:** game state, tick loop, first production chain, map generation, save/load.
