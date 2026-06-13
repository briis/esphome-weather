# ESPHome Weather Display — Project Reference

## What this is

800×480 LVGL v9 weather dashboard on ESP32-S3 using ESPHome. Danish UI ("Vejret i Holmeenegen").
Design: 3×3 dark card grid on pure black background — the 6 px gaps between cards act as visual borders (no explicit borders on cards).

## Current phase

**Phase 2 — Hardware confirmed.** Grid layout verified working on Waveshare ESP32-S3-Touch-LCD-7. SDL simulator used for visual iteration; hardware target flashed via USB and confirmed. Next: populate card content with real HA sensor data.

## Files

| File | Purpose |
| --- | --- |
| `weatherdisplay-common.yaml` | Shared: fonts + full LVGL layout. Included by both targets. |
| `weatherdisplay-sdl.yaml` | SDL simulator target (`host` platform). Layout iteration. |
| `weatherdisplay.yaml` | ESP32-S3 hardware target. Waveshare 7" 800×480. |
| `weather-sdl.yaml` | Old file, superseded (Ecowitt arc layout, kept for reference). |
| `fonts/materialdesignicons-webfont.ttf` | MDI icon font. |

## How to run

SDL simulator (layout iteration):

```sh
esphome run weatherdisplay-sdl.yaml
```

Prerequisites: `brew install sdl2` + `pip install esphome`

Flash to hardware:

```sh
esphome run weatherdisplay.yaml
```

## Layout structure

```text
┌──────────────────────────────────────────────────────────────────┐
│  STATUS BAR  800×44  (wifi icon left · title center · time right)│
├───────────────────┬───────────────────┬──────────────────────────┤
│  card_temperature │    card_wind      │     card_forecast        │
│  (Temperatur)     │    (Vind)         │     (Vejrudsigt)         │
├───────────────────┼───────────────────┼──────────────────────────┤
│  card_rain        │  card_barometer   │     card_sun_uv          │
│  (Regn)           │  (Barometer)      │     (Sol & UV)           │
├───────────────────┼───────────────────┼──────────────────────────┤
│  card_extra_1     │  card_extra_2     │     card_extra_3         │
│  (placeholder)    │  (placeholder)    │     (placeholder)        │
└───────────────────┴───────────────────┴──────────────────────────┘
```

Grid container: x=6, y=50, w=788, h=424. Columns: FR(1)×3. Rows: FR(2), FR(2), FR(1). Gap: 6 px.
Rows 0–1 are roughly twice the height of row 2 (placeholder strip).

Each named card (row 0–1) has a FLEX COLUMN layout with `flex_align_cross: CENTER` and `scrollbar_mode: "OFF"`:

1. Title label (`font_sm`, white, `pad_bottom: 2`)
2. 1 px separator obj (230 px wide, `${clr_separator}`, centered)
3. Content area — card-specific, see Widget IDs section below

## Color palette

Colors are defined as `substitutions:` at the top of `weatherdisplay-common.yaml`. Change the hex value there to retheme globally; individual target files can override any key in their own `substitutions:` block.

| Substitution ID | Hex | Role |
| --- | --- | --- |
| `clr_bg` | `0x000000` | Background |
| `clr_card` | `0x1C2130` | Card background |
| `clr_separator` | `0x2A3348` | Card separator line + status bar border |
| `clr_text` | `0xFFFFFF` | Text primary |
| `clr_text_dim` | `0x7788AA` | Text dim / labels |
| `clr_wifi_good` | `0x4CAF50` | WiFi strong / good (green) |
| `clr_wifi_weak` | `0xFFA726` | WiFi weak (amber) |
| `clr_wifi_bad` | `0xF44336` | WiFi low / off (red) |
| `clr_temp_cold` | `0x2196F3` | Temperature circle — blue (< 5 °C) |
| `clr_temp_cool` | `0xFFC107` | Temperature circle — amber (5–14 °C) |
| `clr_temp_warm` | `0xFF9800` | Temperature circle — orange (15–19 °C) |
| `clr_temp_hot` | `0xF44336` | Temperature circle — red (≥ 20 °C) |

## Fonts

| ID | Family | Size | Use |
| --- | --- | --- | --- |
| `font_xs` | Roboto | 11 | Tiny labels |
| `font_sm` | Roboto | 14 | Default |
| `font_md` | Roboto | 18 | Mid-size values |
| `font_lg` | Roboto | 22 | Status bar (title + time) |
| `font_xl` | Roboto | 32 | Large values |
| `font_xxl` | Roboto | 48 | Hero values |
| `mdi_sm` | MDI | 18 | Small icons |
| `mdi_md` | MDI | 24 | Status bar WiFi icon |

MDI codepoints in use:

| Codepoint | Icon | Use |
| --- | --- | --- |
| `F092B` | wifi-off | No signal / disconnected |
| `F091F` | wifi-strength-1 | Low (< −80 dBm) |
| `F0922` | wifi-strength-2 | Weak (−80 to −67 dBm) |
| `F0925` | wifi-strength-3 | Good (−67 to −55 dBm) |
| `F0928` | wifi-strength-4 | Strong (> −55 dBm) |
| `F092F` | wifi | Generic fallback |
| `F005D` | arrow-up | Max value indicator |
| `F0045` | arrow-down | Min value indicator |
| `F050F` | thermometer | Temperature |
| `F0564` | water | Humidity / rain |
| `F0599` | weather-sunny | Sun / UV |
| `F0597` | weather-rainy | Rain |
| `F059D` | weather-windy | Wind |

## Widget IDs

### Status bar

- `lbl_wifi` — WiFi strength icon (MDI) + colour driven by RSSI. Initial state: wifi-off red. SDL simulates random strength every 30 s; hardware uses `wifi_signal` sensor (60 s) + `on_connect`/`on_disconnect` for immediate feedback.
- `lbl_title` — "Vejret i Holmeenegen" (static)
- `lbl_time` — HH:MM, updated every 60 s via `interval:` in common (uses `::time(nullptr)`). SDL sets correct time on `on_boot`; hardware gets accurate time after `homeassistant` time component syncs (`on_time_sync`).

### Grid cards (row 0)

#### `card_temperature` — outdoor temperature (populated)

Content is a FLEX ROW (`temp_content`) containing three columns:

- **Left** `temp_high_panel` — FLEX COLUMN: red ↑ arrow (`mdi_sm`) + `lbl_temp_high` (`font_md`)
- **Centre** `circle_temp` — 100×100, `radius: 50`, transparent fill, 6 px coloured border. Border colour driven by current temperature threshold. Contains `lbl_temp_current` (`font_lg`, centered).
- **Right** `temp_low_panel` — FLEX COLUMN: blue ↓ arrow (`mdi_sm`) + `lbl_temp_low` (`font_md`)

HA sensors (defined in `weatherdisplay.yaml`, entity IDs set via substitutions overridden there):

| Substitution | Default in common | Override in hardware | Purpose |
| --- | --- | --- | --- |
| `ha_temp_current` | `sensor.unknown_temperature` | `sensor.holmeengen_sensor_temperature` | Current temp + circle colour |
| `ha_temp_high` | `max_temperature_today` | `max_temperature_today` | Daily high (attribute of current) |
| `ha_temp_low` | `min_temperature_today` | `min_temperature_today` | Daily low (attribute of current) |

SDL `on_boot` sets mock values: 12,5 ° current (amber circle), 18,3 ° high, 5,2 ° low.

#### `card_wind` — wind speed/direction (placeholder)

#### `card_forecast` — weather forecast (placeholder)

### Grid cards (row 1)

- `card_rain` — rain stats
- `card_barometer` — pressure
- `card_sun_uv` — solar radiation + UV index

### Grid cards (row 2 — placeholder)

- `card_extra_1`, `card_extra_2`, `card_extra_3`

## Hardware config (weatherdisplay.yaml)

Board: Waveshare ESP32-S3-Touch-LCD-7 · 16 MB flash · 8 MB octal PSRAM

| Component | Detail |
| --- | --- |
| Display platform | `mipi_rgb`, model `ESP32-S3-TOUCH-LCD-7-800X480` |
| Display id | `lvgl_display` |
| Pixel clock | GPIO7, 16 MHz, inverted |
| DE / HSYNC / VSYNC | GPIO5 / GPIO46 / GPIO3 |
| I/O expander | `ch422g` (id: `io_ex`) — display reset/enable, touch reset |
| Touch | GT911 on I2C (SDA GPIO8, SCL GPIO9), interrupt GPIO4 |
| Backlight | `ch422g` io_ex pin 2, `allow_other_uses: true` (shared with display enable) |
| On touch | Resumes LVGL if paused; turns backlight on if off |
| OTA | Turns backlight off on begin (`on_begin: switch.turn_off: backlight`) |
| Captive portal | Enabled (`captive_portal:`) |

### Package strategy

`weatherdisplay.yaml` (hardware) pulls `weatherdisplay-common.yaml` from the **remote Forgejo repo** at `https://git.digitalhomes.dk/briis/esphome-weather` (ref: `main`, refresh: 10 s, auth via `!secret forgejo_token`).

`weatherdisplay-sdl.yaml` (SDL simulator) still uses a local `!include weatherdisplay-common.yaml`.

This means pushing an update to `weatherdisplay-common.yaml` on `main` propagates to the device on next compile/OTA without reflashing the hardware target.

## Known gotchas

- **`::time(nullptr)` required in lambdas** — the `homeassistant` time component introduces the `esphome::time` namespace, making a bare `time()` call ambiguous. Always use `::time(nullptr)` to reference the C stdlib function.
- **`lv_obj_set_style_text_color` selector** — use `0` (not `LV_PART_MAIN`) for the selector argument in lambdas to avoid header-include issues.
- **`scrollbar_mode: "OFF"` needs quotes** — YAML parses bare `OFF` as boolean `false`. Always quote: `scrollbar_mode: "OFF"`.
- **`pad_column` is not a valid `obj` property** — LVGL FLEX column gap must be achieved via `pad_left`/`pad_right` on child objects (transparent `bg_opa: TRANSP` panels make this invisible).
- **HA sensor substitutions belong in the hardware target** — `ha_temp_*` substitutions are defined as placeholders in `weatherdisplay-common.yaml` but **overridden** in `weatherdisplay.yaml`. Main-file substitutions take precedence over package substitutions in ESPHome. Keep all real HA entity IDs in the hardware file so they are always compiled in regardless of Forgejo state.
- **`weatherdisplay.yaml` must be kept in sync in two places** — local dev copy AND the HA ESPHome addon config folder. The common file only needs to be on Forgejo.

## Phase 2 plan

- [x] ESP32-S3 hardware config — verified with real board config
- [x] Flash to hardware — display and touch confirmed working
- [x] Live clock (`lbl_time`) — updates every 60 s, correct on both SDL and hardware
- [x] WiFi signal icon (`lbl_wifi`) — RSSI-based icon + colour, simulated on SDL
- [x] Named color substitutions — all hex values replaced with `${clr_*}` references in common file
- [x] Temperature card (`card_temperature`) — circle outline + current temp + daily high/low, live HA sensors
- [ ] Wind card (`card_wind`)
- [ ] Forecast card (`card_forecast`)
- [ ] Rain card (`card_rain`)
- [ ] Barometer card (`card_barometer`)
- [ ] Sun & UV card (`card_sun_uv`)
