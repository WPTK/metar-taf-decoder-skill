# ICAO / International & Military Format

For reports that aren't US/FAA format. The giveaways: visibility in **meters** (4-digit), altimeter as **`Q####`** (QNH in hPa), trend forecasts (`NOSIG`/`BECMG`/`TEMPO`), `CAVOK`, and sometimes NATO color states. Trust the tokens over the region prefix.

## Visibility (meters)
Reported as a 4-digit value in meters; convert to km/SM for the user.
- `9999` = **10 km or more** (≥ 6.2 SM).
- `0000` = less than 50 m.
- `1200` = 1,200 m (≈ 0.75 SM). `4000` = 4 km (≈ 2.5 SM). (`meters / 1609 = SM`; `meters / 1000 = km`.)
- A trailing direction (`1200NW`) gives the minimum visibility's bearing. A second group like `1400S 4000N` reports directional variation.
- `M` prefix at automated stations = "less than" (e.g. `M1SM` rare in metric).

## Altimeter (QNH)
- `Q####` = QNH in whole hectopascals. `Q1015` = **1015 hPa**. Add inHg in parentheses (`hPa × 0.02953`): 1015 → 29.97 inHg.
- (`A####` if present is inHg — some countries report both conventions.)

## RVR (meters)
`Rxx/####` in meters (no `FT`). `R12/1000U` = runway 12, RVR 1,000 m, increasing (`U`). `M` = below the measurable minimum (`R25/M0075`), `P` = above the maximum (`R33L/P1500`). Trailing `U`/`D`/`N` = up/down/no-change tendency. `L`/`C`/`R` on the designator = left/center/right.

## Sky / visibility shorthand
- `CAVOK` — **Ceiling And Visibility OK**: replaces visibility, weather, and cloud groups when *all* hold: visibility ≥ 10 km; no cloud below 5,000 ft (or the highest minimum sector altitude) and no CB/TCU; no significant weather. (Not used in US domestic reports.)
- `NSC` — No Significant Cloud (no cloud below 5,000 ft / MSA, no CB/TCU, but CAVOK criteria not fully met).
- `NCD` — No Cloud Detected (automated).
- `SKC` — Sky Clear (manual, no cloud). `CLR` — clear below 12,000 ft (automated, US).
- Cloud type: `CB` cumulonimbus, `TCU` towering cumulus appended to a layer. `///` = amount/type not determinable by the automated sensor.

## Wind in m/s
Some countries (e.g. Russia) report wind in `MPS`. `12012MPS` = 120° at 12 m/s. Convert to knots (`m/s × 1.94`): 12 m/s ≈ 23 kt. `00000MPS` = calm. `ABV49MPS`/`P49MPS` = above 49 m/s.

## Trend forecasts (in a METAR)
ICAO METARs may append a landing trend valid for 2 hours:
- `NOSIG` — no significant change expected in the next 2 hours.
- `BECMG <change>` — a lasting change expected.
- `TEMPO <change>` — temporary fluctuations expected.
- Time qualifiers within a trend: `FM` (from), `TL` (until), `AT` (at). e.g. `BECMG FM1130 TL1230 0350` = becoming, from 11:30Z until 12:30Z, visibility 350 m.
For the Discord block, surface the trend on the bottom line (e.g. `Trend: NOSIG (no significant change next 2 hrs)`).

## NATO military color states
Appended at military aerodromes (e.g. German Bundeswehr `ET...`, RAF `EG...`). They summarize the worse of ceiling/visibility into a single color. The value is the **lower bound** for that color (conditions are at least this good):

| Color | Ceiling (≥) | Visibility (≥) |
|---|---|---|
| `BLU` (Blue) | 2,500 ft | 8 km (5 SM) |
| `WHT` (White) | 1,500 ft | 5 km |
| `GRN` (Green) | 700 ft | 3.7 km |
| `YLO` (Yellow) | 300 ft | 1.6 km |
| `AMB` (Amber) | 200 ft | 0.8 km |
| `RED` (Red) | below 200 ft | below 0.8 km |

- `YLO` is sometimes split `YLO1` (700 ft / 3.7 km) and `YLO2` (300 ft / 1.6 km).
- Two states often appear: the current color, then a trend, e.g. `BLU BLU` (currently Blue) or `BLU TEMPO YLO` (Blue now, temporarily Yellow expected).
- `BLACK` prefixes a color to mean the airfield is **closed for non-weather reasons** (obstruction, etc.), not a weather minimum.

(Lesson: an `ET`-prefixed station with color states is a German military field — `ETHB` is Bückeburg Army Airfield, not a civilian typo. Verify the station and decode the colors as standard NATO, not anomalies.)

## Runway state / contamination group (optional, being retired)
An 8-digit group like `8849//91` (often at the very end). Decode `RRECCDDFF`:
- **Runway** (first 2): runway designator; `88` = all runways; `99` = repeat of last report (no new data).
- **Deposit** (3rd): 0 clear/dry, 1 damp, 2 wet, 3 frost, 4 dry snow, 5 wet snow, 6 slush, 7 ice, 8 compacted snow, 9 frozen ruts, `/` not reported.
- **Extent** (4th): 1 = ≤10%, 2 = 11–25%, 5 = 26–50%, 9 = 51–100%, `/` not reported.
- **Depth** (5th–6th): mm of deposit; `00` = <1 mm; `//` = not measurable/not significant; 91–99 = special snow-depth codes.
- **Friction/braking** (7th–8th): coefficient ×100 (`00`–`90`); 91 poor, 92 medium-poor, 93 medium, 94 medium-good, 95 good, 99 unreliable, `//` not reported.
- `8849//91` = all runways, dry snow, 51–100% covered, depth not measurable, braking poor.

(Many regions replaced this with the Global Reporting Format / RWYCC in 2020–2021, but legacy groups still appear.)

## ICAO region prefixes (for identifying the station)
First letter(s) narrow the region; confirm the specific airport with a lookup. Common ones:

- **K** — contiguous USA. **C** — Canada. **M** — Mexico & Central America. **T** — Caribbean (`TJ` Puerto Rico). **P** — Alaska (`PA`), Hawaii (`PH`), Pacific (`PG` Guam).
- **E** — northern Europe: `EG` UK, `EH` Netherlands, `ED` Germany (civil), **`ET` Germany (military)**, `EB` Belgium, `EK` Denmark, `EN` Norway, `ES` Sweden, `EF` Finland, `EI` Ireland, `EP` Poland, `EL` Luxembourg.
- **L** — southern Europe: `LF` France, `LE` Spain, `LI` Italy, `LP` Portugal, `LG` Greece, `LS` Switzerland, `LO` Austria, `LH` Hungary, `LK` Czechia, `LT` Turkey, `LM` Malta.
- **B** — Iceland, Greenland. **G** — northwest Africa (`GM` Morocco, `GC` Canary Is). **D** — `DA` Algeria, `DT` Tunisia, `DN` Nigeria. **H** — `HE` Egypt, `HK` Kenya. **F** — southern Africa (`FA` South Africa).
- **O** — Middle East: `OE` Saudi Arabia, `OM` UAE, `OT` Qatar, `OB` Bahrain, `OK` Kuwait, `OO` Oman, `OI` Iran, `OR` Iraq, `OJ` Jordan, `OP` Pakistan.
- **U** — Russia & former USSR (`UU` Moscow area, `UK` Ukraine). **V** — South & SE Asia mainland: `VI`/`VA`/`VE`/`VO` India, `VT` Thailand, `VV` Vietnam, `VH` Hong Kong, `VC` Sri Lanka. **W** — maritime SE Asia: `WS` Singapore, `WM` Malaysia, `WI`/`WA` Indonesia. **R** — East Asia: `RJ` Japan, `RK` South Korea, `RC` Taiwan, `RP` Philippines. **Z** — China & Mongolia.
- **Y** — Australia. **N** — New Zealand & South Pacific. **S** — South America (`SA` Argentina, `SB` Brazil, `SC` Chile, `SK` Colombia, `SP` Peru).

When the prefix doesn't pin it down, or you're not certain, web-search the ICAO code to confirm the airport name before decoding.
