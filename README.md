# METAR & TAF Decoder

An Agent Skill that decodes aviation weather, a METAR or SPECI (current observation) or a TAF (forecast), into a clear, field-by-field breakdown. Paste a raw report in from any source, and the skill names every field, converts positions and values to readable form, resolves the ICAO code to a full airport name, and explains what the conditions actually are.

It handles all three coding conventions: US/FAA (statute miles, inHg, `RMK` additive groups), ICAO/international (meters, QNH in hPa, `CAVOK`, trend forecasts), and military stations (NATO color states). By default it returns two things: a plain-language readable decode and a Discord-ready code block.

## Overview

METARs and TAFs are terse, coded strings. A single line packs wind, visibility, runway visual range, present weather, cloud layers, temperature and dewpoint, altimeter, and a tail of remark or change groups that each need a lookup table to read correctly. The remark groups in particular (sea-level pressure, the hourly temperature `T`-group, 6-hour temperature extremes, the 3-hour pressure tendency) are where casual decodes go wrong.

This skill parses every field deliberately rather than pattern-matching from memory, reads the relevant reference table before decoding remark and change groups, converts all times to local (which for me is Eastern time, you'll need to customize the skill.md to your specifics) and all temperatures to Fahrenheit (because I'm not a scientist), and flags operationally significant conditions (thunderstorms, freezing precip, low ceilings, gusts, wind shear, corrected or automated observations).

## Installation

An Agent Skill is a plain-Markdown format: a `SKILL.md` file (decode workflow and lookup tables plus a short YAML header) and optional reference files. It is not tied to any one tool. Nothing in this skill calls a model-specific feature; it is instructions and decode tables, so any agent or harness that loads `SKILL.md` skills can use it.

To install, download or clone this repository and place the `metar-taf-decode/` folder in the skills directory your agent reads. The exact path depends on the tool. Cursor, for example, uses `.cursor/skills/metar-taf-decode/`; other tools use their own skills directory, and the folder contents are identical in every case. Consult your tool's documentation for where it looks for skills. `SKILL.md` has to sit at the top level of the `metar-taf-decode/` folder, not nested deeper (`skills/metar-taf-decode/SKILL.md`, not `skills/metar-taf-decode/something/SKILL.md`). One folder too many is the usual install mistake.

Once the folder is in place, the agent loads the skill on its own when a report matches the description, with nothing to invoke manually.

## Activation

The skill triggers when the user wants to decode, read, interpret, or explain a METAR, SPECI, or TAF, pastes raw observation or forecast text, or asks for the weather at an airport by ICAO code. It also triggers on a request to format a decoded report for Discord ("make it a discord message"). Any string starting with `METAR`/`SPECI`/`TAF`, or a 4-letter ICAO code followed by a `DDHHMMZ` timestamp, is enough to fire it.

It does **not** handle ACARS / VDL2 datalink messages or aircraft ownership research.

## Structure

```
metar-taf-decode/
├── SKILL.md                      Decode workflow, format detection, time/unit conversion, output templates
├── README.md                     This file
└── references/
    ├── remarks.md                US/FAA RMK additive groups (SLP, T-group, temp extremes, pressure tendency, precip, peak wind)
    ├── weather-phenomena.md      Present and recent weather codes (intensity, descriptors, phenomena, RE-groups)
    ├── taf.md                    TAF structure, FM/BECMG/TEMPO/PROB change groups, wind shear, output templates
    └── international.md          ICAO/international and military format (meters, QNH, CAVOK, trends, NATO color states)
```

### When each reference is read

| Reference | Read when |
|-----------|-----------|
| `remarks.md` | Always, for any METAR with an `RMK` section. The authority for the additive/coded groups: sea-level pressure, the `T`-group, 6-hour temperature extremes, the 3-hour pressure tendency, precip groups, peak wind, wind shift, variable visibility/ceiling, sensor status. |
| `weather-phenomena.md` | Whenever a present or recent weather group is present: the intensity/proximity qualifiers, descriptors (`TS`/`SH`/`FZ`/`BL`), phenomena (`RA`/`SN`/`FG`/`BR`/`HZ`), and all `RE...` recent-weather codes. |
| `taf.md` | Always, when decoding a TAF. Full structure, the `FM`/`BECMG`/`TEMPO`/`PROB` change groups, valid periods, low-level wind shear, NWS vs ICAO conventions, and the TAF output templates. |
| `international.md` | When the report is **not** US/FAA format: meters visibility, `9999`, QNH in hPa (`Q####`), `CAVOK`/`NSC`/`NCD`, trend forecasts in a METAR, NATO military color states, runway-state groups, and the ICAO-region prefix guide for identifying the airport. |

## Decode workflow

1. **Detect the format** (FAA/US, ICAO/international, or military). This sets the units and which fields to expect. A single report can mix conventions, so the actual tokens (`SM` vs meters, `A####` vs `Q####`) win over the region prefix.
2. **Identify the station.** Resolve the 4-letter ICAO code to the full airport name, verifying when unsure rather than guessing from a prefix.
3. **Parse the body left to right**, field by field: wind, visibility, RVR, present weather, sky condition, temperature/dewpoint, altimeter.
4. **Parse the tail.** For a METAR, the `RMK` section via `remarks.md`. For a TAF, the change groups via `taf.md`.
5. **Convert.** All times to local (the raw Zulu group kept in parentheses), all temperatures to Fahrenheit, preferring the precise `T`-group value over the rounded coded temperature when present.
6. **Produce** the readable decode, then the Discord block.

## Output

By default the skill returns two things, in this order:

1. A **readable decode**: plain language, bulleted, with the full airport name and local observation time in the header and a one-line plain-English summary at the end.
2. A **Discord-ready block**: a monospace code block in a standardized layout, with a header emoji chosen from the dominant condition at the field (thunderstorm, snow or freezing precip, rain, obscuration, overcast, broken, scattered, or clear with a day/night variant). It is delivered as raw copyable text inside an outer fence so the triple backticks survive the paste and land as a code block in Discord.

Ask for only one ("just decode it", "just the discord message") and the skill gives only that. For a TAF it uses a document emoji and a period-by-period layout.

## Conventions

- Times are converted to local in 12-hour format with the raw Zulu (`DDHHMMZ`) kept in parentheses. The timestamp is read day-first (`DDHHMMZ`, not the first four digits as a time), and daylight vs standard time is resolved against the observation's own date.
- Temperatures are given in Fahrenheit and distances/heights in imperial; metric is shown alongside only where it helps a pilot (ICAO pressure in hPa, ICAO visibility in km).
- The full airport name is always given with the ICAO code. An uncertain station is looked up, not guessed, because a misidentified station throws off everything downstream.
- Remark groups, weather phenomena, and TAF change groups are decoded from the reference tables, not reconstructed from memory. The documented traps are called out explicitly: `1####`/`2####` are 6-hour temperature extremes (not time intervals), `53008`-style groups are the 3-hour pressure tendency, and `RE...` groups are standard recent-weather codes.
- Operationally significant conditions are flagged (CB/TCU, low ceiling or visibility, gusts, wind shear, freezing precip, thunderstorms, plus `COR` corrected and `AUTO` automated observations), without editorializing beyond what the data shows.

## Format coverage

- **FAA / US**: statute-mile visibility (`10SM`, `1/2SM`, `P6SM`), altimeter in inHg (`A3005` = 30.05 inHg), and the `RMK` additive groups (`SLP`, `T`-groups, 6-hour extremes, pressure tendency). ICAO codes starting `K`, `P`, `PH`, `TJ`.
- **ICAO / international**: meters visibility (`9999` = 10 km or more), QNH in hPa (`Q1015`), `CAVOK`, and trend forecasts (`NOSIG`/`BECMG`/`TEMPO`).
- **Military**: ICAO format plus NATO color states (`BLU`/`WHT`/`GRN`/`YLO`/`AMB`/`RED`) and runway-state groups.

## Standards

The code forms decoded here are defined by:

- **Federal Meteorological Handbook No. 1 (FMH-1)**, *Surface Weather Observations and Reports*: the US METAR/SPECI coding, including the `RMK` additive groups. The companion NWS instructions cover US TAF.
- **ICAO Annex 3**, *Meteorological Service for International Air Navigation*, and **WMO No. 306**, *Manual on Codes*: the international METAR (FM 15) and TAF (FM 51) code forms.
- NATO color-state conventions for military aerodrome reports.
