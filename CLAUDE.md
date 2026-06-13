# ESPHome Weather Display — Project Reference

## What this is
800×480 LVGL v9 weather dashboard on ESP32-S3 using ESPHome. Styled after an Ecowitt-style station display (dark blue theme, circular arcs, wind compass, sun arc, rain stats, barometer).

## Current phase
**Phase 1 — SDL simulator only.** All layout work happens in `weather-sdl.yaml` with dummy data on the `host` platform. No hardware target yet.

## Files
- `weather-sdl.yaml` — the only source file; SDL simulator + full LVGL layout
- `fonts/materialdesignicons-webfont.ttf` — MDI icon font (codepoints listed below)

## How to run
```
esphome run weather-sdl.yaml
```
Prerequisites: `brew install sdl2` + `pip install esphome`

## Layout structure
```
┌─────────────────────────────────────────────────────────────────┐
│  STATUS BAR  800×32  (lbl_date center, lbl_time right)          │
├───────────────────────────────────────┬─────────────────────────┤
│  LEFT PANEL  450×448                  │  RIGHT PANEL  349×448   │
│  ┌──────────────┬──────────────┐      │  indoor_section 349×182 │
│  │outdoor_temp  │wind_container│      │  (arc_indoor_temp       │
│  │_container    │ 215×210      │      │   arc_indoor_humidity)  │
│  │ 210×210      │              │      ├─────────────────────────┤
│  └──────────────┴──────────────┘      │  rain_section 349×160   │
│  stats_row  450×68                    ├─────────────────────────┤
│  sun_section 450×189                  │  baro_section 349×128   │
└───────────────────────────────────────┴─────────────────────────┘
```
Vertical divider at x=450. Left panel starts at y=32 (below status bar).

## Color palette
| Role | Hex |
|---|---|
| Background | `0x0A1628` |
| Status bar bg | `0x0D1F35` |
| Arc track (empty) | `0x1A2B40` |
| Outdoor temp arc | `0xFFD700` |
| Indoor temp arc | `0xFF7744` |
| Indoor humidity arc | `0x4488FF` |
| Wind dot | `0x7ABBDD` |
| Sun arc indicator | `0xFFDD44` |
| Text primary | `0xFFFFFF` |
| Text dim/labels | `0x7788AA` |
| Text secondary | `0xAABBCC` |
| Text min/max | `0xDDDDDD` |

## Fonts
| ID | Family | Size | Use |
|---|---|---|---|
| `font_xs` | Roboto | 11 | Tiny labels |
| `font_xs_bold` | Roboto 700 | 11 | Cardinal letters on compass |
| `font_sm` | Roboto | 13 | Default, small labels |
| `font_md` | Roboto | 16 | Status bar, sub-values |
| `font_lg` | Roboto | 20 | Stats row values |
| `font_xl` | Roboto | 32 | Indoor values, pressure |
| `font_xxl` | Roboto | 48 | Outdoor temp, wind speed |
| `mdi_sm` | MDI | 16 | Arrow icons |
| `mdi_md` | MDI | 24 | Larger icons |

MDI codepoints in use: `F005D` (arrow-up), `F0045` (arrow-down), `F050F` (thermometer), `F0564` (water), `F0599` (weather-sunny), `F0597` (weather-rainy), `F059D` (weather-windy)

## Widget IDs (all named widgets)
### Status bar
- `lbl_date`, `lbl_time`

### Left panel — outdoor temp
- `arc_outdoor_temp` (−20 to 50, gold)
- `lbl_outdoor_temp` (font_xxl)
- `lbl_temp_max`, `lbl_temp_min`

### Left panel — wind compass
- `arc_wind_compass` (meter, full 360°, rotation=270 → N at top)
- `wind_dot` (14×14 circle, positioned via lambda every 30 s)
- `lbl_wind_dir` (e.g. "NE   52°")
- `lbl_wind_speed` (font_xxl, center)
- `lbl_wind_gust`

### Left panel — stats row
- `lbl_feels_like`, `lbl_dewpoint`, `lbl_out_humidity`
- `lbl_wind_avg` (10-min avg direction + speed)
- `lbl_max_gust`

### Left panel — sun section
- `arc_sun_position` (0–100 dome arc, value=% of day elapsed)
- `lbl_sunrise`, `lbl_sunset`, `lbl_solar_rad`, `lbl_uv`

### Right panel — indoor
- `arc_indoor_temp` (0–40, orange), `lbl_indoor_temp`
- `arc_indoor_humidity` (0–100, blue), `lbl_indoor_humidity`

### Right panel — rain
- `lbl_rain_rate`, `lbl_daily_rain`
- `lbl_rain_event`, `lbl_rain_hourly`, `lbl_rain_weekly`, `lbl_rain_monthly`, `lbl_rain_yearly`

### Right panel — barometer
- `lbl_pressure`, `lbl_pressure_trend`

## Wind dot positioning math
Compass container center: (107, 105). Tick ring outer radius=96, tick length=8, so tick mid radius=92. Dot is 14×14 (radius=7).
```
angle_rad = fmod(270 + dir_deg, 360) * PI / 180
x = 107 + 92 * cos(angle_rad) - 7
y = 105 + 92 * sin(angle_rad) - 7
```
Updated every 30 s via `interval` lambda.

## Phase 2 plan (not started)
- Add ESP32-S3 hardware config + display driver for 800×480
- WiFi + HA API integration
- Map HA sensors (meteobridgesql integration) to the widget IDs above
- Replace dummy data / interval lambda with real `sensor:` + `on_value` lambdas
