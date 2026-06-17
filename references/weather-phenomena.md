# Present & Recent Weather Phenomena

Weather groups are built in a fixed order: **intensity/proximity → descriptor → phenomenon**, concatenated with no spaces (e.g. `+TSRA`, `-SHRA`, `VCFG`). Up to three groups may appear, separated by spaces. Decode each part, then state the whole in plain language.

## Intensity / proximity (prefix)
| Code | Meaning |
|---|---|
| `-` | Light |
| (none) | Moderate |
| `+` | Heavy |
| `VC` | In the vicinity — 5–10 SM from the station (US), not at the field. Used with certain phenomena only (e.g. `VCSH`, `VCFG`, `VCTS`). |

For combined precipitation in one group (e.g. `+TSRASN`), the single intensity prefix applies to the **total** precipitation, dominant type first.

## Descriptors
| Code | Meaning |
|---|---|
| `MI` | Shallow (< 6 ft deep) |
| `PR` | Partial (covering part of the aerodrome) |
| `BC` | Patches |
| `DR` | Low drifting (raised < 6 ft) |
| `BL` | Blowing (raised ≥ 6 ft by wind) |
| `SH` | Showers |
| `TS` | Thunderstorm |
| `FZ` | Freezing (supercooled, freezes on contact) |

## Precipitation
| Code | Meaning |
|---|---|
| `DZ` | Drizzle |
| `RA` | Rain |
| `SN` | Snow |
| `SG` | Snow grains |
| `IC` | Ice crystals (diamond dust) |
| `PL` | Ice pellets |
| `GR` | Hail (≥ 1/4 in / 5 mm) |
| `GS` | Small hail / snow pellets (< 1/4 in) |
| `UP` | Unknown precipitation (automated only) |

## Obscurations
| Code | Meaning |
|---|---|
| `BR` | Mist (visibility ≥ 5/8 SM, < 7 SM) |
| `FG` | Fog (visibility < 5/8 SM) |
| `FU` | Smoke |
| `HZ` | Haze |
| `DU` | Widespread dust |
| `SA` | Sand |
| `PY` | Spray |
| `VA` | Volcanic ash |

Note the visibility convention: **`BR` (mist)** is the obscuration when visibility is 5/8 SM or more; **`FG` (fog)** when below 5/8 SM. `FZFG` = freezing fog (temp below 0°C).

## Other phenomena
| Code | Meaning |
|---|---|
| `PO` | Well-developed dust/sand whirls (dust devils) |
| `SQ` | Squall |
| `FC` | Funnel cloud; `+FC` = tornado or waterspout |
| `DS` | Dust storm |
| `SS` | Sandstorm |

## Common combinations (read these as a unit)
- `+TSRA` — heavy thunderstorm with rain.
- `-SHRA` — light rain showers.
- `TSRA` — thunderstorm with (moderate) rain.
- `VCSH` — showers in the vicinity.
- `VCTS` — thunderstorm in the vicinity.
- `FZRA` — freezing rain. `FZDZ` — freezing drizzle. `FZFG` — freezing fog.
- `+SN` / `-SN` — heavy / light snow. `BLSN` — blowing snow. `DRSN` — low drifting snow.
- `SHSN` — snow showers. `TSSN` — thunderstorm with snow.
- `-RA BR` — light rain and mist (two separate groups).
- `+TSRAGR` — heavy thunderstorm with rain and hail.

## Recent weather (RE...) — for the previous hour, not at observation time
`RE` + phenomenon. These are **standard ICAO codes**, not anomalies. They report weather that occurred since the last report (or in the past hour) but is not happening at observation time. Up to three may appear.

| Code | Meaning |
|---|---|
| `RERA` | Recent rain (mod/heavy) |
| `REDZ` | Recent drizzle |
| `RESN` | Recent snow |
| `RERASN` | Recent rain and snow |
| `RESG` | Recent snow grains |
| `REPL` | Recent ice pellets |
| `REFZRA` | Recent freezing rain |
| `REFZDZ` | Recent freezing drizzle |
| `RESHRA` | Recent rain showers |
| `RESHSN` | Recent snow showers |
| `RESHGR` | Recent showers of hail |
| `RESHGS` | Recent showers of small hail/snow pellets |
| `REBLSN` | Recent blowing snow |
| `RETS` | Recent thunderstorm (no precip) |
| `RETSRA` | Recent thunderstorm with rain |
| `RETSSN` | Recent thunderstorm with snow |
| `RETSGR` | Recent thunderstorm with hail |
| `RESS` | Recent sandstorm |
| `REDS` | Recent dust storm |
| `REFC` | Recent funnel cloud (tornado/waterspout) |
| `REVA` | Recent volcanic ash |
| `REUP` | Recent unknown precipitation (automated) |

Example: `RETSRA` = a thunderstorm with rain occurred recently and has since ended.

## No significant weather
- `NSW` — No Significant Weather (used in TAF change groups to cancel a previously forecast phenomenon).
