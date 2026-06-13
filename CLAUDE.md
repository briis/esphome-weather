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

Each named card (row 0–1) has a FLEX COLUMN layout with `flex_align_cross: CENTER`:

1. Title label (`font_sm`, white, `pad_bottom: 2`)
2. 1 px separator obj (230 px wide, `0x2A3348`, centered)
3. Content area (to be added per card in later phases)

## Color palette

| Role | Hex |
| --- | --- |
| Background | `0x000000` |
| Card background | `0x1C2130` |
| Card separator line | `0x2A3348` |
| WiFi strong / good | `0x4CAF50` (green) |
| WiFi weak | `0xFFA726` (amber) |
| WiFi low / off | `0xF44336` (red) |
| Text primary | `0xFFFFFF` |
| Text dim / labels | `0x7788AA` |

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

- `card_temperature` — outdoor temperature
- `card_wind` — wind speed/direction
- `card_forecast` — weather forecast

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

## Phase 2 plan

- [x] ESP32-S3 hardware config — verified with real board config
- [x] Flash to hardware — display and touch confirmed working
- [x] Live clock (`lbl_time`) — updates every 60 s, correct on both SDL and hardware
- [x] WiFi signal icon (`lbl_wifi`) — RSSI-based icon + colour, simulated on SDL
- [ ] Populate card content areas with real HA sensor data (meteobridgesql)
- [ ] Add HA API sensor mappings to `weatherdisplay-common.yaml`
