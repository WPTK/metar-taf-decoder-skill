---
name: metar-taf-decode
description: >
  Use this skill whenever the user wants to decode, read, interpret, or explain a METAR, SPECI, or
  TAF aviation weather report, or pastes raw weather observation/forecast text and wants to know what it
  says. Triggers include: 'decode this METAR', 'decode this TAF', 'what is the weather at an airport ICAO
  code', 'break this down as a discord message', 'make it a discord message', any pasted string starting
  with METAR/SPECI/TAF or matching the pattern of a 4-letter ICAO code followed by a DDHHMMZ timestamp,
  and any request to format a decoded observation or forecast for Discord. Handles FAA/US format (statute
  miles, inHg, RMK additive groups),
  ICAO/international format (meters, QNH hPa, CAVOK, trend forecasts), and military stations (NATO color
  states). Produces both a plain-language readable decode AND a Discord-ready code-block message by default.
  Do NOT use for ACARS/VDL2 datalink messages (use acars-decode) or aircraft ownership research (use
  aircraft-research).
---

# METAR & TAF Decoder

You are decoding aviation weather. The user pastes a raw METAR (current observation) or TAF (forecast) and you produce two things: a clear plain-language breakdown, and a Discord-ready message formatted exactly the way the user has standardized on. Accuracy matters more than speed here. Parse every field deliberately; do not pattern-match from memory.

## What you produce

**By default, produce BOTH outputs, in this order:**
1. A **readable decode** — plain language, bulleted, with a one-line plain-English summary at the end.
2. A **Discord-ready block** — the standardized code-block paste (templates below).

If the user asks for only one ("just the discord message", "decode it" with no format ask), give what they asked for. When they say "make it a discord message like before" or similar, they mean the code-block format in this skill — go straight to it.

## Reference files

Read the relevant file(s) before decoding. Do not decode remark groups or TAF change groups from memory.

| Reference | When to read |
|-----------|-------------|
| `references/remarks.md` | **Always, for any METAR with an `RMK` section.** Contains every additive/remark group (SLP, T-group, 6-hour temp extremes, 3-hour pressure tendency with the full character-code table, precip groups, peak wind, wind shift, variable visibility/ceiling, sensor status). This is where past decode errors lived — read it. |
| `references/weather-phenomena.md` | Whenever a present/recent weather group is present (anything from the `-`/`+`/`VC` qualifiers, descriptors like `TS`/`SH`/`FZ`/`BL`, or phenomena like `RA`/`SN`/`FG`/`BR`/`HZ`, and all `RE...` recent-weather codes). |
| `references/taf.md` | **Always, when decoding a TAF.** Full TAF structure, the `FM`/`BECMG`/`TEMPO`/`PROB` change groups, valid periods, low-level wind shear, NWS vs ICAO conventions, and the TAF output templates. |
| `references/international.md` | When the report is **not** US/FAA format: meters visibility, `9999`, QNH in hPa (`Q####`), `CAVOK`/`NSC`/`NCD`, trend forecasts (`NOSIG`/`BECMG`/`TEMPO` in a METAR), NATO military color states (`BLU`/`WHT`/`GRN`/`YLO`/`AMB`/`RED`), runway state groups, `MPS` wind units, and the ICAO-region prefix guide for identifying the airport. |

## Workflow

1. **Detect the format** — FAA/US, ICAO/international, or military (see below). This decides units and which fields to expect.
2. **Identify the station** — turn the ICAO code into the full airport name (see Airport identification).
3. **Parse the body** left to right, field by field (see METAR body fields, or `references/taf.md` for TAF).
4. **Parse the tail** — for METAR, the `RMK` section using `references/remarks.md`. For TAF, the change groups using `references/taf.md`.
5. **Convert** all times to Eastern local and all temperatures to Fahrenheit (see Time and Units).
6. **Produce** the readable decode, then the Discord block.

## Detecting the format

- **FAA / US format**: visibility in statute miles with `SM` (e.g., `10SM`, `1/2SM`), altimeter as `A` + 4 digits in inHg (`A3005` = 30.05 inHg), and an `RMK` section with `SLP`/`T`-groups. ICAO codes starting with `K` (CONUS), `P` (Alaska/Pacific), `PH` (Hawaii), `TJ` (Puerto Rico).
- **ICAO / international format**: visibility in meters as a 4-digit value (`1200`, `9999` = 10 km+), altimeter as `Q` + 4 digits in hPa (`Q1015`), trend forecasts (`NOSIG`, `BECMG`, `TEMPO`), `CAVOK`. Read `references/international.md`.
- **Military**: often ICAO format plus NATO color states appended (`BLU`, `YLO`, etc.). German Bundeswehr fields start `ET`. Read `references/international.md` for color-state decoding. (Lesson: `ET` is German military, not a typo — verify the station rather than guessing.)

A single report can mix conventions; trust the actual tokens (`SM` vs meters, `A####` vs `Q####`) over the region prefix.

## Time conversion (Zulu → Eastern)

The user always wants the observation/issuance time in **Eastern local time**, with the raw Zulu kept in parentheses. Get this right; it has been a repeated source of errors.

**The timestamp format is `DDHHMMZ`** — 2-digit day of month, then 4-digit time HHMM, then `Z`. `071156Z` = day **7**, **11:56** UTC. Do **not** read the first four digits as the time. `110256Z` = day 11, 02:56 UTC.

**Convert UTC → Eastern:**
- **EDT (UTC−4)**: from the 2nd Sunday of March through the 1st Sunday of November (roughly mid-March to early November).
- **EST (UTC−5)**: the rest of the year.
- If unsure which side of the boundary today falls on, check the current date with the time tool. Use the **observation's** date, not necessarily today's, if they differ.

Worked examples:
- `110256Z` in June (EDT): 02:56 UTC − 4h = 22:56 the previous day → **10:56 PM EDT, day 10**.
- `091730Z` in July (EDT): 17:30 − 4 = 13:30 → **1:30 PM EDT, day 9**.
- `151200Z` in January (EST): 12:00 − 5 = 07:00 → **7:00 AM EST, day 15**.

Show local time in 12-hour format with AM/PM. Keep the raw Zulu group alongside it.

## Units

The user uses Fahrenheit and imperial. Convert accordingly; when the source is metric, show the converted value as primary and keep the original where it aids a pilot.

| Element | Rule |
|---|---|
| Temperature / dewpoint | Convert °C → °F: `F = C×9/5 + 32`. Prefer the precise `T`-group value (tenths °C) when present; the coded body value (e.g. `23/20`) is rounded. e.g. T-group 24.4°C → **75.9°F**. |
| Wind | Keep in knots and degrees true, add the cardinal direction in parens (e.g. `170° @ 5 kt (S)`). `00000KT` = **Calm**. Gust `G` → "gusting". `VRB` = variable. `dddVddd` = direction varying between the two values. For `MPS` reports, convert to knots (`1 m/s = 1.94 kt`). |
| Visibility (US) | Statute miles as written (`10 SM`, `1/2 SM`). `P6SM` = greater than 6 SM. `M1/4SM` = less than 1/4 SM. |
| Visibility (ICAO) | Meters → show km with SM in parens. `9999` = **≥10 km (6.2+ SM)**. `0000` = less than 50 m. (`meters / 1609 = SM`.) |
| Cloud height | Hundreds of feet AGL × 100. `FEW030` = few at **3,000 ft**, `BKN250` = broken at **25,000 ft**. |
| Altimeter (US) | `A####` is inHg with implied decimal: `A3005` = **30.05 inHg**. |
| Altimeter (ICAO) | `Q####` is QNH in hPa: `Q1015` = **1015 hPa**; add inHg in parens (`hPa × 0.02953`, so 1015 → 29.97 inHg). |
| Sea-level pressure | `SLPppp` is reported in mb/hPa; keep it in mb. Decode rule and the 3-hour tendency are in `references/remarks.md`. |

## METAR body fields (quick reference)

Decode left to right. Anything below that needs a lookup table is in the reference files.

- **Report type**: `METAR` (routine hourly) or `SPECI` (special, off-schedule, conditions changed).
- **Station**: 4-letter ICAO. Resolve to full airport name.
- **Date/time**: `DDHHMMZ` (see Time conversion).
- **Modifier**: `AUTO` = fully automated, no human. `COR` = corrected observation (supersedes a prior one — flag it). Absent when a human observer is logged on.
- **Wind**: `dddssKT` or `dddssGggKT`. Direction true (nearest 10°), speed knots, optional gust. `00000KT` = calm. A separate `dddVddd` group = variable direction.
- **Visibility**: see Units. May be followed by RVR.
- **RVR**: `Rxx/####FT` (US) — runway-specific visual range. `R28L/2600FT` = runway 28 Left, 2,600 ft. `P` = more than, `M` = less than; trailing `V` shows a variable range; trailing `U`/`D`/`N` = up/down/no-change tendency. (ICAO RVR is in meters — see `references/international.md`.)
- **Present weather**: read `references/weather-phenomena.md`. Intensity (`-` light, none = moderate, `+` heavy, `VC` vicinity) + optional descriptor + phenomenon, e.g. `+TSRA` = heavy thunderstorm with rain, `-SHRA` = light rain showers, `BR` = mist, `HZ` = haze.
- **Sky condition**: `SKC`/`CLR` (clear), `FEW`, `SCT` (scattered), `BKN` (broken), `OVC` (overcast), each + 3-digit height in hundreds of feet. `CB` = cumulonimbus, `TCU` = towering cumulus appended to a layer (operationally significant — flag it). `VV###` = vertical visibility into an obscured sky. The **ceiling** is the lowest BKN or OVC layer.
- **Temperature/dewpoint**: `TT/TT` in °C, `M` prefix = below zero (`M06` = −6°C). Convert to °F; refine with the `T`-group if present.
- **Altimeter**: see Units.
- **RMK**: everything after is remarks — read `references/remarks.md`.

## TAF

A TAF is a forecast, not an observation. It opens with `TAF` (or `TAF AMD`/`TAF COR`), the station, an issuance time `DDHHMMZ`, and a **valid period** `DDHH/DDHH`. The base line gives the prevailing forecast; change groups (`FM`, `BECMG`, `TEMPO`, `PROB30`/`PROB40`) modify it for sub-periods. **Read `references/taf.md`** for the full treatment, the change-group rules, low-level wind shear, and the TAF output templates. Convert every period's start/end to Eastern local. Watch the date rollover (`FM100000` is 0000Z on the 10th, not 1000Z).

## Airport identification

Always give the full airport name alongside the ICAO code (the user relies on this). If you know it confidently (e.g. KJAX = Jacksonville International Airport), state it. If you are not certain of the airport for a given ICAO, look it up rather than guessing — the ICAO-region prefix guide in `references/international.md` helps narrow it, and a quick web search confirms it. Misidentifying a station throws off everything downstream, so verify when unsure.

## Output: readable format

Lead with a header line, then bullets for each decoded element, then a remarks line if present, then a one-line plain-English summary. Times in Eastern, temps in °F.

```
**<Full Airport Name> (<ICAO>)** — observed <local time>, <month day>

- Wind: <direction + cardinal> at <speed> kt<, gusting X>
- Visibility: <value>
- Sky: <layers in plain language>
- Temp/dewpoint: <°F> / <°F> (note T-group precision vs rounded coded value if relevant)
- Altimeter: <inHg or hPa(+inHg)>

Remarks: <plain-language decode of the RMK groups — SLP, pressure tendency, temp extremes, etc.>

<One-line plain-English summary of the conditions.>
```

Flag `COR` (corrected) and `AUTO` (automated) where present, and call out any operational hazard (CB, low ceiling/visibility, gusts, freezing precip, thunderstorm).

## Output: Discord format

This is the standardized format the user pastes into Discord. It is a **plain code block** (monospace) — not Discord bold/markdown. Build it like this:

```
[emoji per condition rule below] <ICAO> — <FULL AIRPORT NAME IN CAPS>
Obs: <local time> (<DDHHMMZ>)<  — AUTO / ⚠️ COR if applicable>

Wind: <ddd° @ ss kt> (<cardinal>)
Visibility: <value>
Temperature: <°F> | Dewpoint: <°F>
Altimeter: <inHg, or hPa (inHg) for ICAO>

Clouds: <layers in plain language>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pressure: <SLP mb> (<tendency, e.g. rising +0.8 mb/3hr>)
<extra remark lines as needed: 6-hr max/min, peak wind, etc.>
```

**Critical paste mechanic:** the message must reach the user as **raw copyable text**, including the triple backticks, so it lands as a code block when they paste it. Do not let the Discord block render inside the chat (rendered markdown gets stripped to plain text on paste). Wrap the whole thing in an **outer four-backtick fence** so the inner ```` ``` ```` is visible and selectable. After it, tell them to copy the whole thing including the backticks. Concretely, the response contains:

````
```
🌤️ KJAX — JACKSONVILLE INTL AIRPORT
Obs: 10:56 PM EDT (110256Z)

Wind: 170° @ 5 kt (S)
Visibility: 10 SM
Temperature: 75.9°F | Dewpoint: 68.0°F
Altimeter: 30.03 inHg

Clouds: Few @ 3,000 ft

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pressure: 1017.0 mb (rising +0.8 mb/3hr)
```
````

Format notes:
- **Header emoji always reflects the dominant condition at the field** — there is no fixed default; derive it every time. Choose the **first** that applies, top to bottom:
  - ⛈️ — thunderstorm (`TS` in any form, including `VCTS`)
  - 🌨️ — snow or freezing/frozen precip (`SN`, `SG`, `PL`, `GS`, `GR`, `FZRA`, `FZDZ`, `BLSN`)
  - 🌧️ — rain or drizzle (`RA`, `DZ`, `SHRA`, `-RA`, etc.)
  - 🌫️ — obscuration cutting visibility (`FG`, `BR`, `HZ`, `FU`, `DU`, `SA`, `VA`)
  - ☁️ — no precip/obscuration but an overcast layer (`OVC`)
  - 🌥️ — a broken ceiling (`BKN`) and no `OVC`
  - 🌤️ — only few/scattered cloud (`FEW`/`SCT`)
  - ☀️ / 🌙 — clear (`SKC`/`CLR`/`NSC`/`CAVOK`): use ☀️ if the observation's local time is daytime (about 6 AM–8 PM), 🌙 at night
  Vicinity-only phenomena (`VC…`) don't override the field's own sky state — note them in text instead. The emoji maps to sky cover by the densest layer present, regardless of its height.
- The divider is a row of heavy horizontal box-drawing characters (`━`), about 32 wide.
- For **ICAO/international** reports: visibility line shows km + SM, altimeter shows hPa with inHg in parens, and the bottom section shows the **Trend** (`NOSIG`, `BECMG`, `TEMPO`) instead of an SLP/tendency line when there's no RMK. Flag `AUTO` on the Obs line.
- Keep `COR` visible with a ⚠️ so it's obvious the observation was corrected.
- Only include remark lines that exist in the report. Don't pad with empty fields.

For a **TAF** Discord block, use the document emoji 📋 and the period-by-period layout in `references/taf.md`.

## Decode discipline (avoid these documented mistakes)

- **`1####` and `2####` in remarks are 6-hour temperature extremes, not time intervals.** `10233` = 6-hour **maximum** 23.3°C (→ 73.9°F); `20189` = 6-hour **minimum** 18.9°C (→ 66.0°F). The leading `1` means max, `2` means min. They appear at/near synoptic hours (00/06/12/18 UTC; the `:56` ob like `1156Z` is the 1200Z synoptic observation). Never label these "10-minute" or "recent" values. Full rule in `references/remarks.md`.
- **Read the timestamp as `DDHHMMZ`.** Day first, then time. `071156Z` is 11:56 UTC on the 7th.
- **`53008`-style groups are the 3-hour pressure tendency.** First digit after `5` is the character code (0–3 = net higher than 3h ago, 4 = steady, 5–8 = net lower); last three digits are the change in tenths of a mb. Table in `references/remarks.md`.
- **`RETSRA` and other `RE...` groups are standard recent-weather codes** (`RE` + phenomenon), not anomalies. See `references/weather-phenomena.md`.
- **Verify the station if you're not certain** of the airport. Don't assume a prefix.

## Important rules

- Output temperatures in Fahrenheit, distances/heights imperial; show metric alongside only where it helps a pilot (ICAO pressure, ICAO visibility).
- Always spell out the full airport name with the ICAO code.
- Always convert times to Eastern local with the raw Zulu kept in parentheses.
- Read the relevant reference file before decoding remarks, weather phenomena, TAF change groups, or any ICAO-specific field. Don't reconstruct these tables from memory.
- Call out operationally significant conditions (CB/TCU, low ceiling or visibility, gusts, wind shear, freezing precip, thunderstorms, corrected obs) but don't editorialize beyond what the data shows.
- The Discord block must be delivered as raw copyable text (outer fence), never pre-rendered.
