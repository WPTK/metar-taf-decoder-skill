# TAF Decode Reference

A TAF (Terminal Aerodrome Forecast) is a forecast for a ~5 SM radius of an aerodrome, valid 24 or 30 hours, issued (typically) every 6 hours. Structure: header → base forecast → change groups that modify the base for sub-periods. Decode the base, then walk each change group in order. Convert every time to Eastern local (see SKILL.md Time conversion); keep the raw Zulu groups in parentheses.

## Header
- **Type**: `TAF` (routine), `TAF AMD` (amended — supersedes the prior TAF), `TAF COR` (corrected). Flag AMD/COR.
- **Station**: 4-letter ICAO → full airport name.
- **Issuance time**: `DDHHMMZ` (day, time, Zulu). `091730Z` = issued 17:30Z on the 9th.
- **Valid period**: `DDHH/DDHH` — start `DDHH`, end `DDHH`. `0918/1024` = valid from 18:00Z on the 9th to 24:00Z on the 10th (i.e. 00:00Z on the 11th). 24- or 30-hour span. **An hour of `24` means end-of-day / 00Z the next day.**

## Base forecast line
Same field types as a METAR body, but **forecast** values: wind, visibility, weather, clouds, and (US) optional low-level wind shear. No temperature/dewpoint, no altimeter, no remarks.
- Visibility: `P6SM` = greater than 6 SM (US TAF for "unlimited-ish"). ICAO uses `9999`/`CAVOK`.
- Clouds: same as METAR (`FEW`/`SCT`/`BKN`/`OVC` + height). In a TAF, only **`CB`** is appended for cloud type (no `TCU`). `VV###` for vertical visibility.
- `NSW` = No Significant Weather (used in change groups to cancel previously forecast weather).

### Low-level wind shear (WS group)
`WShhh/dddssKT` — non-convective low-level wind shear (≤ 2,000 ft AGL).
- `hhh` = height of the shear layer in hundreds of feet. `/`. `ddd` `ss` = wind direction/speed **above** that height, in knots.
- `WS010/31022KT` = at 1,000 ft, wind 310° at 22 kt (shear between surface and 1,000 ft). Flag it; it's operationally significant for departures/approaches.

## Change groups

Each group describes how conditions change from what preceded it. They appear in chronological order.

### FM — FROM  (`FMDDHHMM`)
A **rapid, lasting** change at a specific time. Everything after `FM` **replaces** the entire previous forecast from that time forward (a new prevailing condition), until the next `FM` or the end of the valid period. Note the **6-digit** time: day, hour, **minute**.
- `FM091930` = from 19:30Z on the 9th.
- **Date-rollover trap:** `FM100000` = 00:00Z on the **10th**, not 10:00Z. `FM100100` = 01:00Z on the 10th.

### BECMG — BECOMING  (`BECMG DDHH/DDHH`)
A **gradual** change expected to become established at some point during the window and then **persist**. The window `DDHH/DDHH` is the transition period; treat the new conditions as prevailing from the end of the window onward.
- `BECMG 1013/1015` = becoming, over 13:00Z–15:00Z on the 10th.
- Only the elements that change are listed; unlisted elements carry over.
- (NWS domestic TAFs use `BECMG` but exclude temperature forecasts.)

### TEMPO — TEMPORARY  (`TEMPO DDHH/DDHH`)
**Temporary fluctuations** within the window, each lasting < 1 hour and in total covering < half the period. Conditions revert to the prevailing forecast between fluctuations — they are not a lasting change.
- `TEMPO 0920/0922` = temporarily between 20:00Z and 22:00Z on the 9th.

### PROB30 / PROB40  (`PROB30 DDHH/DDHH`)
A **30% or 40% probability** of the described conditions occurring in the window. (Only 30 and 40 are used; higher/lower certainty is expressed differently.) Often pairs with `TEMPO`. **NWS does not use PROB in the first 9 hours** of a TAF.
- `PROB30 1004/1007` = 30% chance, 04:00Z–07:00Z on the 10th.

## Regional / format notes
- **NWS (US domestic) TAFs**: visibility `P6SM`/`SM`, `SKC`/`FEW` rather than `CAVOK`; exclude `BECMG` temperature groups; no `PROB` in the first 9 hours.
- **ICAO TAFs**: meters/`9999`/`CAVOK`, QNH context, may include `TX`/`TN` temperature groups (`TXmm/DDHHZ` max, `TNmm/DDHHZ` min).
- **US military TAFs**: add turbulence and icing groups.
- For non-US TAFs, read `references/international.md` for `CAVOK`, meters, and color states.

---

## Worked example (the FAA card TAF)

```
TAF KPIT 091730Z 0918/1024 15005KT 5SM HZ FEW020 WS010/31022KT
     FM091930 30015G25KT 3SM SHRA OVC015
     TEMPO 0920/0922 1/2SM +TSRA OVC008CB
     FM100100 27008KT 5SM SHRA BKN020 OVC040
     PROB30 1004/1007 1SM -RA BR
     FM101015 18005KT 6SM -SHRA OVC020
     BECMG 1013/1015 P6SM NSW SKC
```
Decodes to (assuming EDT): issued 1:30 PM EDT on the 9th, valid 2:00 PM EDT the 9th → 8:00 PM EDT the 10th. Base: SSE 5 kt, 5 SM in haze, few at 2,000 ft, with low-level wind shear (310° @ 22 kt at 1,000 ft). From 3:30 PM: WNW 15 kt gusting 25, 3 SM in rain showers, overcast 1,500 ft. Temporarily 4:00–6:00 PM: 1/2 SM in heavy thunderstorm with rain, overcast 800 ft cumulonimbus. From 9:00 PM (the 9th): W 8 kt, 5 SM rain showers, broken 2,000 / overcast 4,000. 30% chance midnight–3:00 AM: 1 SM in light rain and mist. From 6:15 AM: S 5 kt, 6 SM light rain showers, overcast 2,000. Becoming 9:00–11:00 AM: greater than 6 SM, no significant weather, clear.

---

## Output: readable TAF

```
**<Full Airport Name> (<ICAO>)** — TAF issued <local time>, <month day>
Valid: <start local> → <end local>

Base (from <start local>):
- Wind / Visibility / Sky / (Wind shear if present)

From <local time>:
- <changed elements>

Temporary <local window>:
- <fluctuating elements>

PROB30 <local window>:
- <possible elements>

Becoming <local window>:
- <new prevailing elements>

<One- or two-line plain-English summary of how the day shapes up — the headline hazards and trend.>
```
Flag AMD/COR, wind shear, and convective/IFR periods (thunderstorms, low ceilings/visibility).

## Output: Discord TAF

Header emoji **📋**. One labeled block per period; reuse the SKILL.md divider. Plain code block, delivered raw inside an outer four-backtick fence (same paste mechanic as METAR).

```
📋 <ICAO> — <FULL AIRPORT NAME IN CAPS>
TAF issued <local time> (<DDHHMMZ>)<  — AMD / COR if applicable>
Valid: <start local> → <end local>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BASE — from <start local>
Wind: <ddd° @ ss kt> (<cardinal>)<  gusting gg>
Visibility: <value><  in <weather>>
Clouds: <layers>
Wind shear: <ddd° @ ss kt at hhh ft>   ← only if present

FROM <local time>
Wind / Visibility / Clouds (changed elements)

TEMPO <local window>
<fluctuating elements>

PROB30 <local window>
<possible elements>

FROM <local time>
...

BECMG <local window>
<new prevailing elements>
```
Show only the elements each group actually changes. Keep `CB` flagged on cloud lines. Convert all windows to Eastern local.
