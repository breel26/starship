# Open Questions

Every unresolved design question so far, grouped by topic. Each has an ID (Q1, Q2…) so you can answer by number, e.g. "Q4: no minimum time". Where I've made a suggestion, it's noted; answered questions move to ARCHITECTURE.md and are removed from this list.

Last updated: 18 Oct 2026.

---

## Research & technology

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q1 | **Minimum research time per tech?** Right now enough RP could unlock Nuclear Fission in a single tick. | A per-tech minimum (e.g. Fission ≥ 1 year) gives extra RP diminishing returns. Tunable in `balance.json`. | tech-tree-physics.md |
| Q2 | **Starting knowledge:** should the 1890s discoveries (X-rays, Electron, Radioactivity) already be unlocked at the start, or stay as quick early research? | Current draft: quick early research. | tech-tree-physics.md |
| Q3 | **RP costs and pacing:** a steady 2 RP/tick finishes the physics tree in ~17 game years vs 38 real years. Is that the right pace, or closer to history? | Start as drafted; tune after play-testing. | tech-tree-physics.md |
| Q4 | **Lab output rule:** RP = base × staffing × funding (capped at 100%). Keep, or add over-funding and diminishing returns? | | ARCHITECTURE §10 |
| Q5 | **Discovery chances** per lab and university, and how big a "lump" of RP a discovery gives. | Starting value: 25% of the tech's cost. | ARCHITECTURE §10 |
| Q6 | **Great-scientist bonuses:** career length and bonus sizes (draft: +10–20% to one or two fields for ~9 game years). | | tech-tree-physics.md |
| Q7 | **Other fields' trees:** chemistry and medicine example trees, plus engineering/computing fields for later eras. | Draft chemistry next, same format as physics. | — |

## Eras & progression

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q8 | **What does "the current era" mean** now that eras don't gate research? A label, a milestone with rewards, or something that changes housing, buildings or events? | e.g. the era advances when you complete a set of key techs, with a celebration screen and new events. | ARCHITECTURE §14 |
| Q9 | **Win and progress conditions:** is launching the first (non-FTL) starship a win, a milestone, or the start of the space game? | | — |

## Scheduling & construction

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q10 | **Crowding curve:** how much does each worker past a crew's max still help? | Each extra worker adds half as much as the last. | ARCHITECTURE §11 |
| Q11 | **Overhead rates:** site cost per tick while an activity is open. | | ARCHITECTURE §11 |
| Q12 | **Partial material delivery:** can work start before all materials arrive, at a reduced rate? | Yes, work proceeds in proportion to materials on site. | ARCHITECTURE §11 |
| Q13 | **Relationship types in the UI:** show all four (FS, SS, FF, SF) from the start, or Finish-to-Start first? | FS by default, others under "advanced". | ARCHITECTURE §11 |
| Q14 | **Planner progression:** which techs or hires improve the built-in planner, and what each unlocks (leveling, critical path, etc.). | Gantt Charts (1910s), CPM (1957), Computers. | ARCHITECTURE §11 |

## Transport, shipping & worlds

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q15 | **Shipping between worlds:** do ships exist as units you build and route, or is it an abstract route system? How is travel time set? | | ARCHITECTURE §8 |
| Q16 | **Galaxy size:** how many star systems and planets? | | ARCHITECTURE §6 |
| Q17 | **FTL starship mechanic:** how FTL works and its versions. *Deferred to a future update.* | Nothing now assumes slower-than-light. | ARCHITECTURE §8 |
| Q18 | **Roads for commutes:** when should worker commutes start using the road network (movement stage 3)? | | ARCHITECTURE §7 |

## Time & saves

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q19 | **Offline progress:** does the game keep running while it's closed? | Options: no; yes, capped (e.g. 8 hours); yes, fully simulated at high speed on return. | ARCHITECTURE §14 |

## Economy

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q20 | **Value window:** 100 ticks to start. Confirm after play-testing. | Tunable in `balance.json`. | ARCHITECTURE §4 |
| Q21 | **Wages and food:** how much GORE a worker costs per tick, and how food is produced and distributed. | | properties.md |

## Art & UI

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q22 | **Who makes the final art?** Hand-drawn sprites (you, a hired artist, an asset pack) or procedurally generated like the mockups? | Generated placeholders first, replaced over time. | — |
| Q23 | **Schedule screen mockup:** Gantt chart and the activity panel with the Resources tab. | Next mockup. | — |
| Q24 | **Other screen mockups:** Construction, Research (tech tree), Logistics, Stockpiles, People, Star Map. | | — |
| Q25 | **Zoomed-out map:** what the map looks like at the 12 km view, where 64 px art is too small to read. | Simplified overview tiles. | ARCHITECTURE §6 |

## Development

| ID | Question | Suggestion / notes | Source |
|---|---|---|---|
| Q26 | **When to start coding:** keep planning, or scaffold milestone 1 (Vite + TypeScript + Phaser 4) now? | | ARCHITECTURE §15 |
