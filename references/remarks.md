# METAR Remarks (RMK) Decode Reference

Everything after `RMK` in a US/FAA METAR. Decode every group present. This file is the authority for the additive/coded groups; do not decode them from memory. Groups appear roughly in the order below but are omitted when not applicable.

## Table of contents
- Automated station type (AO1/AO2)
- Peak wind (PK WND)
- Wind shift (WSHFT)
- Tower/surface visibility, variable visibility
- Lightning
- Begin/end of precipitation & thunderstorms
- Variable ceiling, second-location ceiling
- Pressure rising/falling rapidly (PRESRR/PRESFR)
- Sea-level pressure (SLPppp)  ← common
- Hourly precipitation (Prrrr)
- 3-/6-hour precipitation (6RRRR)
- 24-hour precipitation (7R24...)
- Hourly temp/dewpoint (T-group)  ← common, gives precision
- 6-hour max temp (1snTTT)  ← FREQUENT ERROR SOURCE
- 6-hour min temp (2snTTT)  ← FREQUENT ERROR SOURCE
- 24-hour max/min temp (4-group)
- 3-hour pressure tendency (5appp)  ← FREQUENT ERROR SOURCE, full code table
- Sensor status indicators
- Maintenance indicator ($)

---

## Automated station type
- `AO1` — automated station **without** a precipitation discriminator (can't tell rain from snow).
- `AO2` — automated station **with** a precipitation discriminator. Most US ASOS sites.

## PK WND — Peak wind
`PK WND dddff(f)/(hh)mm`: direction (tens of degrees), speed (knots), and time since the last METAR.
- `PK WND 20032/25` = peak wind from 200° at 32 kt, at :25 past the hour.
- `PK WND 28045/1547` = from 280° at 45 kt at 15:47Z (4-digit time when the hour is included).

## WSHFT — Wind shift
`WSHFT (hh)mm`: time a wind shift began. `WSHFT 1715` = wind shift began at 17:15Z. May carry `FROPA` if the shift was due to a frontal passage (`WSHFT 30 FROPA`).

## Visibility remarks
- `TWR VIS v` — visibility observed from the control tower. `TWR VIS 2` = 2 SM from the tower.
- `SFC VIS v` — surface (ASOS) visibility.
- `VIS vVv` — **variable** prevailing visibility (reported when < 3 SM and variable). `VIS 3/4V1 1/2` = varying between 3/4 and 1 1/2 SM.
- `VIS v [LOC]` — visibility at a secondary location. `VIS 3/4 RWY11` = 3/4 SM at runway 11.

## Lightning
`[FREQ] LTG [type] [LOC]`: frequency + lightning + location.
- Frequency: `OCNL` occasional, `FRQ` frequent, `CONS` continuous.
- Type: `IC` in-cloud, `CC` cloud-cloud, `CG` cloud-ground, `CA` cloud-air.
- Example: `FRQ LTG CG NE` = frequent cloud-to-ground lightning to the northeast.

## Begin/end of precip & thunderstorms
`w'w'B(hh)mmE(hh)mm` and `TSB(hh)mmE(hh)mm`.
- `RAB07` = rain began at :07. `RAE45` = rain ended at :45. `RAB07E45` = began :07, ended :45.
- `TSB0159E30` = thunderstorm began 01:59Z, ended :30.
- `SHRAB25` = showers of rain began at :25.

## Ceiling remarks
- `CIG hhhVhhh` — **variable** ceiling (reported when ceiling < 3,000 ft and variable). `CIG 013V017` = ceiling varying between 1,300 and 1,700 ft.
- `CIG hhh [LOC]` — ceiling at a secondary sensor. `CIG 017 RWY11` = 1,700 ft at runway 11.

## PRESRR / PRESFR
Pressure **rising** or **falling** rapidly at the time of observation. No value attached; it's a flag.

## SLPppp — Sea-level pressure  (common)
Sea-level pressure in hectopascals (= millibars), given as tens, units, and tenths with the leading `9` or `10` dropped. Reconstruct by choosing the prefix that lands the value nearest 1013:
- If `ppp` begins with **5–9**, prefix **9** → `9pp.p`.
- If `ppp` begins with **0–4**, prefix **10** → `10pp.p`.

Examples: `SLP177` → **1017.7 mb**. `SLP045` → **1004.5 mb**. `SLP232` → **1023.2 mb**. `SLP882` → **988.2 mb**. `SLP125` → **1012.5 mb**.

Report SLP in mb (don't convert to inHg). `SLPNO` = sea-level pressure not available.

## Prrrr — Hourly precipitation
Liquid-equivalent precip since the last METAR, in hundredths of an inch. `P0003` = 0.03 in. `P0000` = a trace.

## 6RRRR — 3- and 6-hour precipitation
Liquid-equivalent precip in hundredths of an inch. The **period depends on the observation hour**:
- **6-hour** amount at the 00, 06, 12, 18 UTC observations.
- **3-hour** amount at the 03, 09, 15, 21 UTC observations.
- `60009` = 0.09 in over the period. `60000` = a trace.

## 7R24R24R24R24 — 24-hour precipitation
Liquid-equivalent precip over the past 24 hours, in hundredths of an inch, reported in the 12 UTC observation. `70015` = 0.15 in in 24 hours.

## T-group — Hourly temperature & dewpoint  (gives precise values)
`TsnTaTaTasnTdTdTd`: more precise temp/dewpoint in tenths of a degree Celsius.
- After `T`: sign digit (`0` = ≥ 0°C, `1` = below 0°C) + 3 digits = temperature in tenths °C.
- Then: sign digit + 3 digits = dewpoint in tenths °C.
- `T02330200` = temp **+23.3°C**, dewpoint **+20.0°C**. → 73.9°F / 68.0°F.
- `T10171028` = temp **−1.7°C**, dewpoint **−2.8°C**.

Prefer these over the rounded body values (e.g. `23/20`) for the output; note the rounding if it differs.

## 1snTxTxTx — 6-HOUR MAXIMUM temperature  (do not mislabel)
The leading **`1`** marks a 6-hour **maximum** temperature.
- Sign digit (`0` = ≥ 0°C, `1` = below 0°C) + 3 digits = max temp in tenths °C.
- `10233` = 6-hour maximum **23.3°C** → **73.9°F**.
- `10066` = 6-hour maximum **6.6°C**.
- Reported at the 00/06/12/18 UTC synoptic observations (the `:56` ob preceding them, e.g. `1156Z`, carries the 1200Z data).
- **This is NOT a "10-minute" value and NOT a "recent" value.** Past decodes have made exactly this mistake.

## 2snTnTnTn — 6-HOUR MINIMUM temperature  (do not mislabel)
The leading **`2`** marks a 6-hour **minimum** temperature. Same encoding as the max group.
- `20189` = 6-hour minimum **18.9°C** → **66.0°F**.
- `21012` = 6-hour minimum **−1.2°C** (sign digit `1` = below zero).
- Reported at the 00/06/12/18 UTC synoptic observations.
- **Not a "20-minute" value.** The `2` means minimum, not 20 minutes.

Mnemonic: in the 6-hour temp groups, the leading digit is the **kind** (1 = max, 2 = min), not a duration.

## 4snTxTxTxsnTnTnTn — 24-hour max/min temperature
Reported once daily at the local-midnight observation.
- First sign+3 digits = 24-hour maximum; next sign+3 digits = 24-hour minimum (tenths °C).
- `400461006` → max +4.6°C, min −0.6°C (note the digit count: `4` + `0046` + `1006`).

## 5appp — 3-HOUR PRESSURE TENDENCY  (full code table)
`5` = group indicator. `a` = character of pressure change over the past 3 hours (0–8). `ppp` = magnitude of change in tenths of a millibar (hPa).

**Net direction:** codes **0–3 = higher** than 3 hours ago, **4 = same**, **5–8 = lower** than 3 hours ago.

| `a` | Meaning | Net result |
|---|---|---|
| 0 | Increasing, then decreasing | Higher than 3h ago |
| 1 | Increasing, then steady; or increasing more slowly | Higher than 3h ago |
| 2 | Increasing steadily or unsteadily | Higher than 3h ago |
| 3 | Decreasing or steady, then increasing; or increasing more rapidly | Higher than 3h ago |
| 4 | Steady | Same as 3h ago |
| 5 | Decreasing, then increasing | Lower than 3h ago |
| 6 | Decreasing, then steady; or decreasing more slowly | Lower than 3h ago |
| 7 | Decreasing steadily or unsteadily | Lower than 3h ago |
| 8 | Steady or increasing, then decreasing; or decreasing more rapidly | Lower than 3h ago |

Examples:
- `53011` → character 3 (rising, net higher), change **1.1 mb** over 3 hours.
- `53008` → character 3 (rising, net higher), change **0.8 mb**. Plain language: "pressure rose 0.8 mb over the past 3 hours, now higher than 3 hours ago."
- `58033` → character 8 (steady/increasing then decreasing rapidly, net lower), change **3.3 mb**.
- `52032` → character 2 (steady increase), change **3.2 mb**.

For the Discord block, phrase as e.g. `(rising +0.8 mb/3hr)` or `(falling −3.3 mb/3hr)`. Use the magnitude with a sign reflecting the net direction (0–3 → `+`, 5–8 → `−`, 4 → "steady").

## Sensor status indicators
Reported when an automated sensor is inoperative:
- `RVRNO` — RVR missing. `PWINO` — present-weather identifier not operating. `PNO` — precip accumulation gauge not operating. `FZRANO` — freezing-rain sensor not operating. `TSNO` — thunderstorm/lightning sensor not operating. `VISNO [LOC]` / `CHINO [LOC]` — visibility / ceiling sensor at a secondary location not operating.

## $ — Maintenance check indicator
A trailing `$` means the automated system has detected it needs maintenance. Note it; it doesn't invalidate the observation.

---

## Other remark strings (plain-language, not coded)
Manual/augmented remarks may appear in plain language, e.g.:
- `TORNADO B25 N MOV E` — tornado began :25, north of the field, moving east.
- `VIRGA` — precipitation not reaching the ground.
- `FROPA` — frontal passage.
- `CB DSNT NW` — cumulonimbus distant to the northwest. `DSNT` = distant (> ~10 SM); `VC` = vicinity (5–10 SM).
- `ACFT MSHP` — aircraft mishap. `FIRST`/`LAST` — first/last ob around a break in coverage.
Decode these literally using the abbreviation list; they aren't numeric groups.
