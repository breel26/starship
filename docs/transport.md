# Transport by Era — Reviewed

Every route and vehicle has an ID (R = route, V = vehicle) so you can comment by number, e.g. "V14: too slow" or "drop R9".

**Speeds** are typical *operating* averages for the year listed (what a real journey averaged), not top speeds or records. Capacities are typical for the period. Items marked 🔮 are speculative or still in development in the real world, so their numbers are best estimates.

**Eras** are the game's 9 eras (see ARCHITECTURE.md). Years mark when the item first became practical in the real world.

**Expandable:** each era's routes and vehicles live in that era's `routes.json` and `vehicles.json`. New routes, vehicles or whole eras are added as new entries; nothing here needs editing to add more. New IDs continue the sequence (R33, V73, X5…).

**Older vehicles stay useful:** nothing is retired when a new era arrives. Horses can work alongside maglev trains; older vehicles just end up doing a smaller share of the work.

**Why real speeds work:** at 1000x, one in-game year passes in about 2 seconds, so even real-world trip times (a 7-month Mars transfer, a 40-year interstellar voyage) stay playable.

---

## Era 1 — Steam (1900–1919)

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R1 | Trail / footpath | — | Free to place; foot, pack animals, riders |
| R2 | Dirt wagon road | — | Wagons and early cars; slow and muddy when wet |
| R3 | Gravel (macadam) road | 1900 | Faster, all-weather wagons, early cars and trucks |
| R4 | Freight railroad | 1900 | Steam trains; needs stations and coal/water stops |
| R5 | Streetcar tracks | 1900 | Electric trams in towns; needs power plant |
| R6 | Subway | 1904 | Underground city rail (NYC opened 1904); expensive |
| R7 | River dock / harbor | — | Boats and ships load here |
| R8 | Canal | — | Barges; very cheap per ton, very slow |

### Vehicles
| ID | Vehicle | Year | Speed (km/h) | Carries | Needs |
|---|---|---|---|---|---|
| V1 | On foot | — | 5 | 1 person, ~20 kg | any |
| V2 | Pack mule | — | 5 | ~90 kg | R1+ |
| V3 | Riding horse | — | 10 | 1 person | R1+ |
| V4 | Horse-drawn freight wagon | — | 4 | ~1.5 t | R2+ |
| V5 | Stagecoach / carriage | — | 10 | 6–9 people | R2+ |
| V6 | Bicycle | 1900 | 15 | 1 person | R2+ |
| V7 | Steam traction engine | 1900 | 6 | ~15 t (towed) | R2+ |
| V8 | Early automobile (Model T) | 1908 | 35 | 4–5 people | R2+ (faster on R3) |
| V9 | Early motor truck | 1912 | 20 | ~3 t | R3+ |
| V10 | Steam freight train | 1900 | 30 | ~1,500 t | R4 |
| V11 | Steam passenger train | 1900 | 60 | ~300 people | R4 |
| V12 | Electric streetcar | 1900 | 15 | ~50 people | R5 |
| V13 | Subway train | 1904 | 25 | ~400 people | R6 |
| V14 | Canal barge (horse-towed) | — | 4 | ~100 t | R8 |
| V15 | River steamboat / barge tow | — | 10 | ~1,000 t | R7 |
| V16 | Sailing schooner | — | 13 | ~500 t | R7 |
| V17 | Steam cargo ship | 1900 | 17 | ~5,000 t | R7 |
| V18 | Ocean liner | 1907 | 45 | ~2,000 people | R7 |
| V19 | Airship (early Zeppelin) | 1910 | 80 | ~20 people or ~2 t | airship mast |
| V20 | Early biplane | 1914 | 110 | 1–2 people | grass airfield |

---

## Era 2 — Diesel (1920–1944)

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R9 | Paved road (asphalt/concrete) | 1920 | Cars and trucks at full speed |
| R10 | Highway | 1926 | Numbered long-distance paved roads, at-grade crossings |
| R11 | Freeway (limited access) | 1935 | No crossings or traffic lights (Autobahn 1935, PA Turnpike 1940) |
| R12 | Interurban electric railway | 1920 | Light rail linking towns |
| R13 | Airfield with paved runway | 1930 | Needed for airliners |
| R14 | Airship terminal | 1928 | Mooring mast + hangar |

### Vehicles
| ID | Vehicle | Year | Speed (km/h) | Carries | Needs |
|---|---|---|---|---|---|
| V21 | Passenger car | 1925 | 60 | 4–5 people | R9+ |
| V22 | Bus | 1925 | 50 | ~35 people | R9+ |
| V23 | Motor truck (pneumatic tires) | 1925 | 50 | ~7 t | R9+ |
| V24 | Crawler tractor | 1925 | 8 | ~20 t (towed, off-road) | any |
| V25 | Diesel-electric freight locomotive | 1935 | 50 | ~3,000 t | R4 |
| V26 | Streamliner passenger train | 1934 | 100 | ~200 people | R4 |
| V27 | Interurban railcar | 1920 | 40 | ~60 people | R12 |
| V28 | Diesel motor ship | 1925 | 22 | ~10,000 t | R7 |
| V29 | Large airship (Graf Zeppelin) | 1928 | 115 | ~50 people or ~10 t | R14 |
| V30 | Prop airliner (DC-3) | 1936 | 300 | ~25 people or ~2.5 t | R13 |
| V31 | Flying boat (Clipper) | 1935 | 250 | ~40 people | R7 |

---

## Era 3 — Jet (1945–1969)

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R15 | Interstate freeway network | 1956 | Freeways linking whole regions |
| R16 | Container port | 1956 | Cranes + container yards; huge loading speedup |
| R17 | Jet airport | 1958 | Long runways, terminals |
| R18 | Heliport | 1950 | Small pad, anywhere |
| R19 | High-speed rail line | 1964 | Dedicated track (Shinkansen) |
| R20 | Rocket launch pad | 1957 | Launches to orbit |

### Vehicles
| ID | Vehicle | Year | Speed (km/h) | Carries | Needs |
|---|---|---|---|---|---|
| V32 | Car (freeway) | 1956 | 100 | 4–5 people | R11+ |
| V33 | Semi tractor-trailer | 1950 | 85 | ~20 t | R10+ |
| V34 | Long-range prop airliner (Constellation) | 1947 | 500 | ~80 people | R13+ |
| V35 | Jet airliner (707) | 1958 | 900 | ~150 people | R17 |
| V36 | Cargo turboprop (C-130 class) | 1956 | 540 | ~20 t | R13+ |
| V37 | Helicopter | 1950 | 200 | ~10 people or ~2 t | R18 |
| V38 | Early container ship | 1956 | 30 | ~20,000 t | R16 |
| V39 | Oil tanker / bulk carrier | 1960 | 28 | ~100,000 t | R7 |
| V40 | Bullet train (Shinkansen) | 1964 | 210 | ~1,000 people | R19 |
| V41 | Hovercraft | 1968 | 100 | ~250 people + 30 cars | R7 / beach |
| V42 | Expendable rocket | 1957 | to orbit in ~10 min | ~1–100 t to orbit (late 1960s) | R20 |

---

## Era 4 — Information (1970–1999)

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R21 | Modern light rail | 1978 | Street-level urban rail, cheaper than subway |
| R22 | Mega container terminal | 1980 | Automated cranes, giant ships |
| R23 | Spaceport | 1981 | Reusable vehicle landing + processing |

### Vehicles
| ID | Vehicle | Year | Speed (km/h) | Carries | Needs |
|---|---|---|---|---|---|
| V43 | Wide-body jet (747) | 1970 | 900 | ~400 people or ~110 t | R17 |
| V44 | Supersonic transport (Concorde) | 1976 | 2,150 | ~100 people | R17 |
| V45 | High-speed train (TGV) | 1981 | 270 | ~500 people | R19 |
| V46 | Light rail vehicle | 1978 | 30 | ~200 people | R21 |
| V47 | Large container ship | 1988 | 45 | ~60,000 t | R22 |
| V48 | Space Shuttle | 1981 | orbit in ~9 min | ~25 t to orbit | R23 |

---

## Era 5 — Modern (2000–2029)

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R24 | Maglev line | 2004 | Magnetic levitation track (Shanghai 2004) |
| R25 | Drone port | 2020 | Small automated delivery hub |

### Vehicles
| ID | Vehicle | Year | Speed (km/h) | Carries | Needs |
|---|---|---|---|---|---|
| V49 | Maglev train | 2004 | 430 | ~600 people | R24 |
| V50 | 350 km/h high-speed train | 2008 | 320 | ~1,000 people | R19 |
| V51 | Electric / autonomous truck | 2025 | 90 | ~20 t | R10+ |
| V52 | Cargo drone | 2020 | 80 | ~5 kg | R25 |
| V53 | Ultra-large container ship | 2010 | 35 | ~200,000 t | R22 |
| V54 | Reusable rocket (Falcon 9 class) | 2015 | orbit in ~9 min | ~20 t to orbit | R23 |
| V55 | Super-heavy reusable rocket 🔮 | 2025 | orbit in ~9 min | ~100+ t to orbit | R23 |
| V56 | Electric air taxi (eVTOL) 🔮 | 2026 | 200 | ~4 people | R25 |

---

## Era 6 — Near Future (2030–2069) 🔮

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R26 | Long-distance maglev | 2035 | Intercity superconducting maglev (Chuo Shinkansen planned) |
| R27 | Vacuum tube line | 2045 | Low-pressure tube, sealed track |
| R28 | Hypersonic airport | 2045 | Specialized fuel + heat-resistant handling |
| R29 | Suborbital launch site | 2040 | Point-to-point rocket travel on one planet |

### Vehicles
| ID | Vehicle | Year | Speed (km/h) | Carries | Needs |
|---|---|---|---|---|---|
| V57 | Long-distance maglev | 2035 | 500 | ~1,000 people | R26 |
| V58 | Vacuum tube train | 2045 | 1,000 | ~30 people per pod, frequent | R27 |
| V59 | Hypersonic transport (Mach 5) | 2045 | 5,500 | ~50 people | R28 |
| V60 | Suborbital point-to-point rocket | 2040 | ~27,000 (anywhere on Earth in <1 h) | ~100 people or ~100 t | R29 |
| V61 | Autonomous truck convoy | 2030 | 90 | ~100 t (convoy) | R11+ |

---

## Era 7 — Orbital (2070–2119) 🔮

### Routes
| ID | Route | Year | Notes |
|---|---|---|---|
| R30 | Orbital station | 2070 | Transfer hub in orbit |
| R31 | Mass driver | 2080 | Electromagnetic cargo launcher (best on airless moons) |
| R32 | Space elevator | 2100 | Cable from surface to geostationary orbit |

### Vehicles
| ID | Vehicle | Year | Trip time | Carries | Needs |
|---|---|---|---|---|---|
| V62 | Orbital tug | 2070 | hours between orbits | ~50 t | R30 |
| V63 | Lunar shuttle (chemical) | 2070 | Earth → Moon ~3 days | ~50 t | R23 / R30 |
| V64 | Mass driver cargo pod | 2080 | to orbit in minutes, cargo only | ~1 t per shot, many shots | R31 |
| V65 | Space elevator climber | 2100 | surface → GEO ~7.5 days (200 km/h) | ~20 t, very cheap per ton | R32 |

---

## Era 8 — Interplanetary (2120–2199) 🔮

| ID | Vehicle | Year | Trip time (Earth → Mars) | Carries | Needs |
|---|---|---|---|---|---|
| V66 | Chemical transfer ship | 2120 | ~7–9 months; launch windows every ~26 months | ~100 t | R30 |
| V67 | Nuclear thermal ship | 2130 | ~4 months | ~200 t | R30 |
| V68 | Ion / solar-electric cargo hauler | 2130 | ~12–18 months, very cheap | ~1,000 t | R30 |
| V69 | Fusion torch ship | 2180 | ~1 month, no launch windows | ~500 t | R30 |

---

## Era 9 — Interstellar (2200+) 🔮

| ID | Vehicle | Year | Trip time (to nearest star, ~4.2 ly) | Carries | Needs |
|---|---|---|---|---|---|
| V70 | Light-sail probe (20% light speed) | 2200 | ~20 years | grams; scouting only | laser array |
| V71 | Fusion colony ship (~5% light speed) | 2250 | ~85 years | colonists + cargo | R30 |
| V72 | **The Starship** (game goal) | — | ? | ? | ? |

---

## Also worth considering (not on your list)

| ID | Item | Year | Why |
|---|---|---|---|
| X1 | Pipelines (oil, gas, water) | 1900 | Moves fluids continuously; big logistics shortcut |
| X2 | Aerial ropeway / cable car | 1900 | Moves ore down mountains; fits snowy-peak terrain |
| X3 | Conveyor belts | 1905 | Short-range bulk movement inside industrial areas |
| X4 | Pneumatic tubes | 1900 | Small items and messages within cities |

---

## Decisions

1. **Map scale:** 100 m tiles. The view shows at most 12 km and pans; the map starts county-sized (~40 km) and grows with exploration until it can cover a whole world. Fast vehicles get real distances to cover on-map.
2. **The Starship can be faster than light.** Its mechanic will be designed in a future update; V72 stays a placeholder until then.
3. **Older vehicles stay useful** and do a smaller share of the work as newer ones arrive.
4. **Eras** stay as listed here.
