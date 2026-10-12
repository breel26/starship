# Object Properties — Reviewed

Every property has an ID so you can comment by number, e.g. "M7: use 0–10 instead" or "drop P16".

The goal is a deep simulation: objects carry realistic properties from day one, even before any game system uses them, so the simulation can grow later without reworking the data.

---

## Rules

1. **Units are in the field name.** `massKg`, `speedKmh`, `meltingPointC`. All units are metric/SI. No unitless numbers except 0–1 scales, which are marked.
2. **Everything optional has a default.** An entry only lists what matters for it; the schema fills in the rest. Only a few fields per type are required.
3. **Derived properties.** Objects list what they're made of (`composition`); weight, value, fire resistance and similar are computed from the materials at load time. Any derived value can be overridden by writing it explicitly; explicit always wins.
4. **Real-world values,** consistent with `transport.md`.
5. **Dormant fields are fine.** A field can exist before any system reads it. The **Used** column shows when:
   - **Now** — first milestones
   - **Soon** — planned systems (logistics, food, durability, vehicles)
   - **Later** — expected expansions (fire, power grid, pollution, health)
   - **Future** — possible deep-simulation features
6. **Validated.** Every field has an allowed range checked at load (e.g. `hardnessMohs` 1–10). Bad data fails loudly.
7. **Value and energy are in GORE.** All prices, power, energy, wages and food use GORE (see "Value in GORE" below).
8. **Expandable.** New properties are added with a default, so existing JSON keeps working. `tags` (S7) allows ad-hoc labels without a schema change.

---

## Value in GORE

**GORE** (Geosynchronous ORbital Energy) is the game's currency and energy unit.

- **1 GORE = 13,800 kcal = 57.7 MJ**, the energy needed to lift 1 kg from the surface to geosynchronous orbit.
- **Power is measured in GORE per tick** (one 8-hour shift). **1 GORE/tick ≈ 2.69 hp ≈ 2.0 kW**; 1 hp ≈ 0.372 GORE/tick.
- **Value = mass + manufacturing energy + work premium:**
  - **Mass:** 1 GORE per kg.
  - **Manufacturing energy** ("embodied energy"): the energy spent making it, added up through every step of its production chain. Raw materials found in nature have none.
  - **Work premium:** if it can do work, its power in GORE/tick × 100. The 100-tick **value window** is one tunable number in the data, to adjust after play-testing.
- **Things that run for minutes, not ticks** (rockets, engine burns): the premium is the energy they can actually deliver within the value window.
- **Unpowered things** have no work premium: raw iron ore is worth just its mass (1 GORE/kg); a wagon is worth its mass plus the energy used to build it.
- **How manufacturing energy is worked out:** each recipe run uses energy = (building power + workers' work output) × duration. The outputs are worth the value of their inputs plus that energy, split across the outputs. The game calculates this at load by walking the recipe chain, so authors don't type it in; it can be overridden per item.
- **Real-world reference** (typical energy to produce 1 kg, for authoring recipes): concrete ≈ 0.02, brick ≈ 0.05, sawn timber ≈ 0.15, glass ≈ 0.26, steel ≈ 0.4, copper ≈ 1, plastic ≈ 1.4, aluminum ≈ 3 GORE/kg. So 1 kg of aluminum ends up worth about 4 GORE, versus about 1.4 for steel.

| Example | Mass | Power | Value (mass + work; manufacturing energy comes from recipes) |
|---|---|---|---|
| Iron ore, 1 kg | 1 kg | — | 1 GORE (raw, no manufacturing energy) |
| Person | 70 kg | 0.04 GORE/tick (0.1 hp) | 74 GORE |
| Horse | 500 kg | 0.37 GORE/tick (1 hp) | 537 GORE |
| Model T | 540 kg | 7.4 GORE/tick (20 hp) | 1,284 GORE |
| Cat 335 excavator | 34,900 kg | 101.5 GORE/tick (273 hp) | 45,054 GORE |

Per-vehicle values are in `transport.md`.

---

## Shared — every object (S)

| ID | Field | Type / unit | Default | Used |
|---|---|---|---|---|
| S1 | `id` | lowercase id | required | Now |
| S2 | `name` | text | required | Now |
| S3 | `era` | era id | required (except materials found in nature) | Now |
| S4 | `description` | text | "" | Now |
| S5 | `icon` | sprite id | same as `id` | Now |
| S6 | `valueGore` | GORE | derived: `massKg` + manufacturing energy + `powerGorePerTick` × value window | Now |
| S7 | `tags` | list of text | [] | Now |
| S8 | `massKg` | kg | derived from `composition` | Soon |
| S9 | `sizeM` | [length, width, height] m | none | Soon |
| S10 | `volumeM3` | m³ | derived from `sizeM`, or mass ÷ density | Soon |

---

## Materials (M) — iron, wood, coal, glass, grain…

| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| M1 | `form` | bulk / packaged / liquid / gas / discrete | packaged | Soon | which storage and vehicles can carry it |
| M2 | `densityKgM3` | kg/m³ | **required** | Soon | weight, volume, cargo limits |
| M3 | `hardnessMohs` | 1–10 | 1 | Later | mining speed, tool wear |
| M4 | `tensileStrengthMPa` | MPa | 0 | Later | construction, vehicle design |
| M5 | `compressiveStrengthMPa` | MPa | 0 | Later | foundations, tall buildings |
| M6 | `meltingPointC` | °C | none (doesn't melt) | Soon | smelting temperatures, furnace tiers |
| M7 | `ignitionPointC` | °C | none (non-flammable) | Later | fire spread, storage risk |
| M8 | `energyGorePerKg` | GORE/kg | 0 | Now | fuel value (coal ≈ 0.42, diesel ≈ 0.78) |
| M9 | `thermalConductivityWmK` | W/m·K | 1 | Future | insulation, heat |
| M10 | `electricalConductivitySm` | S/m | 0 | Future | wiring, power grid |
| M11 | `corrosionResistance` | 0–1 | 0.5 | Later | durability, rust |
| M12 | `toxicity` | 0–1 | 0 | Later | health, pollution |
| M13 | `radioactivityBqKg` | Bq/kg | 0 | Future | radiation, nuclear era |
| M14 | `perishShifts` | shifts | none (doesn't spoil) | Soon | food spoilage |
| M15 | `rarity` | 0–1 | 0.5 | Now | deposit generation |
| M16 | `recyclableFraction` | 0–1 | 0 | Later | scrap and recycling |
| M17 | `embodiedGorePerKg` | GORE/kg | derived from recipes (0 for raw materials) | Now | manufacturing energy; part of value |

---

## Items & equipment (I) — tools, machines, parts

| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| I1 | `composition` | { material: kg } | **required** (except raw materials) | Soon | derived mass, value, fire resistance |
| I2 | `durability.maxCondition` | 0–100 | 100 | Soon | wear and breakdown |
| I3 | `durability.wearPerShift` | condition/shift | 0.05 | Soon | how fast it wears out |
| I4 | `maintenance` | { material: amount per 100 shifts } | {} | Soon | spare parts upkeep |
| I5 | `powerGorePerTick` | GORE/tick (negative = consumes from grid) | 0 | Now | work output; adds to value; power grid later |
| I6 | `fuel` | { resource, perShift } | none | Soon | fuel consumption |
| I7 | `heatGorePerTick` | GORE/tick | 0 | Future | heat, comfort |
| I8 | `pollution` | { type: amount per shift } | {} | Later | air/water/soil pollution |
| I9 | `noiseDb` | dB | 0 | Future | neighbor comfort |
| I10 | `fireResistance` | 0–1 | derived from composition | Later | fire damage |
| I11 | `speedMultiplier` | × | 1 | Now | production speed bonus |
| I12 | `lifespanYears` | years | none | Later | aging out of service |

---

## Buildings (B) — have all Items fields, plus:

| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| B1 | `footprintTiles` | [w, h] tiles (100 m each) | [1, 1] | Now | placement; **one building per tile**, except housing |
| B2 | `heightM` | m | 10 | Future | skyline, line of sight |
| B3 | `buildShifts` | shifts | **required** | Now | construction time |
| B4 | `requiredTerrain` | list of terrain ids | any buildable | Now | where it can go |
| B5 | `workerSlots` | [{ role, count }] | [] | Now | jobs |
| B6 | `maxShiftsPerDay` | 1–3 | 3 | Now | night shifts |
| B7 | `housingCapacity` | people per tile | 0 (10 for early housing) | Now | homes; **starts at 10 per tile and rises with later housing types** (apartments, arcologies) |
| B8 | `storageM3` | m³ | 0 | Now | stockpile space |
| B9 | `storageForms` | list of forms (see M1) | [packaged] | Soon | silos vs tanks vs warehouses |
| B10 | `foundationLoadKPa` | kPa | 100 | Later | needs terrain with enough bearing capacity (T6) |
| B11 | `comfort` | 0–1 | 0 | Later | morale of residents/workers |
| B12 | `fireRisk` | 0–1 | derived | Later | fire chance |

---

## Vehicles (VE) — have Items fields, plus (values per vehicle in `transport.md`):

| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| VE1 | `network` | road / rail / water / air / space | **required** | Now | which routes it uses |
| VE2 | `minGrade` | route grade | 1 | Now | which routes are good enough |
| VE3 | `speedKmh` | km/h | **required** | Now | trip time |
| VE4 | `capacity.cargoTonnes` | t | 0 | Now | how much it hauls |
| VE5 | `capacity.cargoM3` | m³ | derived from size | Soon | bulky-cargo limit |
| VE6 | `capacity.passengers` | people | 0 | Soon | moving workers and colonists |
| VE7 | `cargoForms` | list of forms (see M1) | [packaged] | Soon | tanker vs hopper vs flatbed |
| VE8 | `crew` | { worker type: count } | { driver: 1 } | Soon | staffing |
| VE9 | `fuel` | { resource, perKm } | none | Soon | fuel use |
| VE10 | `rangeKm` | km | none (unlimited) | Later | refueling stops |
| VE11 | `emissions` | { type: amount per km } | {} | Later | pollution |
| VE12 | `maxGradePercent` | % slope | 10 | Later | hills and mountains |
| VE13 | `offRoad` | yes/no | no | Soon | travel without routes |
| VE14 | `reliabilityShifts` | average shifts between breakdowns | none | Later | breakdowns |
| VE15 | `weatherSensitivity` | 0–1 | 0 | Future | weather delays |

---

## People (P)

Worker **types** are templates in `workers.json`; each **person** in the game is an instance with their own values.

### Worker type (JSON)
| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| P1 | `skills` | { skill: starting level 0–100 } | {} | Now | who can do which job |
| P2 | `trainingShifts` | shifts | 0 | Soon | time to train into this role |
| P3 | `wageGorePerTick` | GORE/tick | 0 | Soon | labor cost |
| P4 | `needs.foodGorePerTick` | GORE/tick | 0.06 (≈ 2,500 kcal/day) | Soon | food demand. A person eats more energy (0.06/tick) than they put out as work (0.04/tick), as in real life |
| P5 | `needs.waterLPerDay` | L | 3 | Later | water demand |
| P6 | `needs.housing` | beds | 1 | Now | housing demand |
| P7 | `walkKmh` | km/h, **minimum 10** | 10 (riding-horse pace, V3) | Now | commute and travel time when not aboard a vehicle; aboard one (e.g. a wagon), the vehicle's speed applies instead. Values below 10 fail validation |
| P8 | `carryKg` | kg | 25 | Later | hand-carrying goods |

### Person (game state)
| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| P9 | `homeId`, `targetId`, `shift` | ids | none | Now | where they live and work |
| P10 | `skillLevels` | { skill: 0–100 } | from type | Soon | output quality/speed, grows with experience |
| P11 | `name` | text | generated | Soon | personality |
| P12 | `age` | years | 18–60 at random | Later | aging, retirement |
| P13 | `health` | 0–1 | 1 | Later | illness, injuries, pollution effects |
| P14 | `fatigue` | 0–1 | 0 | Later | overtime and night shifts |
| P15 | `morale` | 0–1 | 0.7 | Later | productivity, leaving |
| P16 | `education` | level 0–5 | 0 | Later | which roles they can train for |
| P17 | `traits` | list | [] | Future | personality effects |
| P18 | `householdId` | id | none | Future | families, population growth |

---

## Terrain (T) — adds to the terrain fields in ARCHITECTURE.md

| ID | Field | Type / unit | Default | Used | Drives |
|---|---|---|---|---|---|
| T1 | `buildable` | yes/no | yes | Now | placement |
| T2 | `moveCost` | × walking time | 1 | Now | commutes, off-road travel |
| T3 | `yields` | list of resources | [] | Now | what can be harvested |
| T4 | `deposits` | { resource: chance } | {} | Now | generation |
| T5 | `fertility` | 0–1 | 0 | Soon | farming output |
| T6 | `bearingCapacityKPa` | kPa | 150 | Later | heavy buildings need firm ground |
| T7 | `slopePercent` | % | 0 | Later | vehicle grades (VE12) |
| T8 | `needsDrainage` | yes/no | no | Later | wetlands |
| T9 | `temperatureRangeC` | [min, max] °C | [−10, 30] | Later | crops, comfort, freezing |
| T10 | `precipitationMmYear` | mm/year | 800 | Later | farming, floods |
| T11 | `waterTableM` | m below surface | 10 | Future | wells |

---

## Planets (W) — adds to planet types

| ID | Field | Type / unit | Default (Earth) | Used | Drives |
|---|---|---|---|---|---|
| W1 | `circumferenceKm` | km | 40,000 | Now | world size |
| W2 | `waterCoverage` | 0–1 | 0.7 | Now | generation |
| W3 | `gravityG` | g | 1 | Later | launch costs, vehicle loads |
| W4 | `atmospherePressureAtm` | atm | 1 | Later | breathable? aircraft? |
| W5 | `atmosphere` | { gas: fraction } | Earth air | Future | life support |
| W6 | `surfaceTempC` | [min, max] °C | [−50, 50] | Later | habitability |
| W7 | `radiationMSvYear` | mSv/year | 2.4 | Future | health off Earth |
| W8 | `dayLengthHours` | h | 24 | Future | day/night cycles |
| W9 | `orbitAU` | AU from its star | 1 | Later | shipping trip times |

---

## Example — a material and a vehicle

```json
{
  "id": "iron_plate",
  "name": "Iron Plate",
  "era": "steam",
  "form": "packaged",
  "densityKgM3": 7870,
  "hardnessMohs": 4,
  "tensileStrengthMPa": 350,
  "meltingPointC": 1538,
  "corrosionResistance": 0.2,
  "recyclableFraction": 0.9
}
```

```json
{
  "id": "freight_wagon",
  "name": "Horse-drawn Freight Wagon",
  "era": "steam",
  "network": "road",
  "minGrade": 2,
  "speedKmh": 4,
  "capacity": { "cargoTonnes": 1.5 },
  "composition": { "wood": 350, "iron_plate": 40 },
  "crew": { "teamster": 1 },
  "fuel": { "resource": "fodder", "perKm": 0.5 },
  "offRoad": true
}
```
Derived at load: `massKg` 390, `valueGore` 390 + the manufacturing energy of its wood and iron (unpowered, so no work premium), `fireResistance` low (mostly wood).

---

## Decisions

1. **Buildings:** one building per 100 m tile, except housing. Housing starts at 10 people per tile; later housing types raise the density.
2. **Currency:** GORE (1 GORE = 13,800 kcal = 57.7 MJ). Value = mass + manufacturing energy + 100 ticks of power output (if it does work).
3. **Power:** everything is in GORE per tick.
4. **People:** modeled individually for now, with the option to group large populations later.

## Open questions

1. **Fuel:** should a fuel's stored energy (coal 0.42, diesel 0.78 GORE/kg) add to its value on top of its mass and manufacturing energy?
