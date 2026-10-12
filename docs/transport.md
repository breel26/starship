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
| V1 | On foot | — | 10 (game rule: people move at least at horse pace, see properties P7) | 1 person, ~20 kg | any |
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

## Values in GORE

How values are worked out (see ARCHITECTURE.md §5a):
- **Value = mass in kg + power in GORE/tick × 100** (the 100-tick value window; tunable after play-testing).
- **1 GORE/tick ≈ 2.69 hp ≈ 2.0 kW**; 1 hp ≈ 0.372 GORE/tick. Power is the vehicle's sustained engine output.
- **Unpowered vehicles** (wagons, barges, sailing ships, cargo pods) are worth their mass only; the horses, mules, wind or launcher doing the work are valued separately.
- **Things that run for minutes, not ticks** (rockets, departure burns): the premium is the energy they can actually deliver within the 100-tick window, i.e. the energy of the launches or burns possible in that time.
- **Jet aircraft:** power is estimated as cruise thrust × cruise speed.
- **Rail:** locomotives are valued without their cars. Rolling stock is separate: freight car ~15,000 GORE, passenger coach ~40,000 GORE (mass only).
- Masses and powers are approximate (±30%) real-world figures; 🔮 items are best estimates.
- **These values are mass + work premium only.** Manufacturing energy is added automatically once recipes exist (roughly +40% of mass for steel-built vehicles, more for aluminum aircraft).

| ID | Vehicle | Mass (kg) | Power (GORE/tick) | Value (GORE) | Notes |
|---|---|---|---|---|---|
| V1 | On foot (person) | 70.0 | 0.04 | **73.7** |  |
| V2 | Pack mule | 400 | 0.26 | **426** |  |
| V3 | Riding horse | 500 | 0.37 | **537** |  |
| V4 | Horse-drawn freight wagon | 600 | 0 | **600** | unpowered; horses priced separately |
| V5 | Stagecoach / carriage | 1,100 | 0 | **1,100** | unpowered; horses separate |
| V6 | Bicycle | 18.0 | 0 | **18.0** | rider supplies the power |
| V7 | Steam traction engine | 12,000 | 18.6 | **13,860** |  |
| V8 | Early automobile (Model T) | 540 | 7.4 | **1,284** |  |
| V9 | Early motor truck | 3,000 | 11.2 | **4,116** |  |
| V10 | Steam freight locomotive | 100,000 | 558 | **155,793** | locomotive + tender; cars separate |
| V11 | Steam passenger locomotive | 90,000 | 446 | **134,634** | locomotive + tender; coaches separate |
| V12 | Electric streetcar | 15,000 | 29.8 | **17,976** |  |
| V13 | Subway motor car | 35,000 | 149 | **49,878** | per car (~80 people); train = several cars |
| V14 | Canal barge | 25,000 | 0 | **25,000** | towed by mules/horses |
| V15 | River steam towboat | 300,000 | 186 | **318,598** | barges separate |
| V16 | Sailing schooner | 300,000 | 0 | **300,000** | wind-powered; no engine premium |
| V17 | Steam cargo ship | 2,500,000 | 558 | **2,555,793** |  |
| V18 | Ocean liner | 35,000,000 | 28,268 | **37,826,828** | Lusitania class |
| V19 | Airship (early Zeppelin) | 13,000 | 162 | **29,180** |  |
| V20 | Early biplane | 500 | 29.8 | **3,476** |  |
| V21 | Passenger car (1920s–30s) | 1,000 | 14.9 | **2,488** |  |
| V22 | Bus | 6,000 | 27.9 | **8,790** |  |
| V23 | Motor truck | 5,000 | 22.3 | **7,232** |  |
| V24 | Crawler tractor | 9,000 | 22.3 | **11,232** |  |
| V25 | Diesel-electric freight locomotive | 104,000 | 502 | **154,213** | one unit; cars separate |
| V26 | Streamliner passenger train | 97,000 | 223 | **119,317** | whole articulated trainset |
| V27 | Interurban railcar | 40,000 | 149 | **54,878** |  |
| V28 | Diesel motor ship | 4,000,000 | 1,488 | **4,148,780** |  |
| V29 | Large airship (Graf Zeppelin) | 67,000 | 1,023 | **169,287** |  |
| V30 | Prop airliner (DC-3) | 7,650 | 893 | **96,918** |  |
| V31 | Flying boat (Clipper) | 11,200 | 1,413 | **152,541** |  |
| V32 | Car (1950s) | 1,600 | 55.8 | **7,179** |  |
| V33 | Semi tractor-trailer | 12,000 | 74.4 | **19,439** | tractor + empty trailer |
| V34 | Long-range prop airliner (Constellation) | 25,500 | 3,720 | **397,451** |  |
| V35 | Jet airliner (707) | 66,400 | 10,043 | **1,070,668** | jet power ≈ cruise thrust × speed |
| V36 | Cargo turboprop (C-130 class) | 28,000 | 5,579 | **585,927** |  |
| V37 | Helicopter | 2,260 | 223 | **24,577** |  |
| V38 | Early container ship | 6,000,000 | 2,232 | **6,223,171** |  |
| V39 | Oil tanker / bulk carrier | 18,000,000 | 9,299 | **18,929,878** |  |
| V40 | Bullet train (Shinkansen) | 700,000 | 4,426 | **1,142,622** | 12-car set |
| V41 | Hovercraft (SR.N4) | 165,000 | 5,059 | **670,854** |  |
| V42 | Expendable rocket (Saturn V class) | 182,000 | per use | **381,171** | premium = energy of its one launch |
| V43 | Wide-body jet (747) | 162,000 | 21,573 | **2,319,316** | jet power ≈ cruise thrust × speed |
| V44 | Supersonic transport (Concorde) | 78,700 | 72,530 | **7,331,747** | jet power ≈ cruise thrust × speed |
| V45 | High-speed train (TGV) | 385,000 | 3,217 | **706,738** | whole trainset |
| V46 | Light rail vehicle | 32,000 | 149 | **46,878** |  |
| V47 | Large container ship | 20,000,000 | 18,598 | **21,859,756** |  |
| V48 | Space Shuttle | 286,500 | per use | **468,352** | premium = energy of one launch (~1 per window) |
| V49 | Maglev train (Shanghai) | 300,000 | 4,984 | **798,414** | motors are in the track |
| V50 | 350 km/h high-speed train | 447,000 | 4,389 | **885,902** | 8-car set |
| V51 | Electric / autonomous truck | 17,000 | 186 | **35,598** | tractor + empty trailer |
| V52 | Cargo drone | 10.0 | 1.0 | **110** |  |
| V53 | Ultra-large container ship | 60,000,000 | 29,756 | **62,975,609** |  |
| V54 | Reusable rocket (Falcon 9 class) | 30,000 | per use | **73,298** | premium = energy of one launch (~1 per window) |
| V55 | Super-heavy reusable rocket 🔮 | 300,000 | per use | **757,228** | premium = energy of one launch |
| V56 | Electric air taxi (eVTOL) 🔮 | 1,800 | 74.4 | **9,239** |  |
| V57 | Long-distance maglev 🔮 | 400,000 | 9,968 | **1,396,829** | 16-car set |
| V58 | Vacuum tube train pod 🔮 | 15,000 | 372 | **52,195** |  |
| V59 | Hypersonic transport (Mach 5) 🔮 | 60,000 | 175,933 | **17,653,288** | jet power ≈ cruise thrust × speed |
| V60 | Suborbital point-to-point rocket 🔮 | 300,000 | per use | **757,228** | premium = energy of one launch |
| V61 | Autonomous truck convoy 🔮 | 85,000 | 930 | **177,988** | 5 trucks |
| V62 | Orbital tug 🔮 | 5,000 | 49.8 | **9,984** | electric propulsion, continuous |
| V63 | Lunar shuttle (chemical) 🔮 | 30,000 | per use | **125,256** | premium = ~5 trips per window |
| V64 | Mass driver cargo pod 🔮 | 200 | 0 | **200** | unpowered; the mass driver does the work |
| V65 | Space elevator climber 🔮 | 10,000 | 8,146 | **824,573** | lifting 20 t at 200 km/h |
| V66 | Chemical transfer ship 🔮 | 100,000 | per use | **252,409** | premium = one departure burn |
| V67 | Nuclear thermal ship 🔮 | 150,000 | per use | **212,349** | premium = one departure burn |
| V68 | Ion / solar-electric cargo hauler 🔮 | 50,000 | 4,988 | **548,786** | 10 MW, thrusts continuously |
| V69 | Fusion torch ship 🔮 | 500,000 | 4,984,145 | **498,914,498** | 10 GW drive, continuous |
| V70 | Light-sail probe 🔮 | 0.001 | 0 | **0.001** | the laser array does the work |
| V71 | Fusion colony ship 🔮 | 100,000,000 | — | **100,000,000** | drive premium TBD with interstellar mechanics |
| V72 | The Starship | — | — | TBD | designed with the FTL mechanic |

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
