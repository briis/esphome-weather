# ESPHome Weather Display — Project Reference

## What this is

800×480 LVGL v9 weather dashboard on ESP32-S3 using ESPHome. Danish UI ("Vejret i Holmeenegen").
Design: 3×3 dark card grid on pure black background — the 6 px gaps between cards act as visual borders (no explicit borders on cards).

## Current phase

**Phase 1 — Layout iteration.** SDL simulator (`weatherdisplay-sdl.yaml`) for visual work. Hardware file (`weatherdisplay.yaml`) is ready with verified config for the Waveshare ESP32-S3-Touch-LCD-7. Next: flash to hardware and wire up real sensor data.

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
| WiFi icon (active) | `0x4CAF50` |
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

MDI codepoints in use: `F092F` (wifi), `F005D` (arrow-up), `F0045` (arrow-down),
`F050F` (thermometer), `F0564` (water), `F0599` (weather-sunny),
`F0597` (weather-rainy), `F059D` (weather-windy)

## Widget IDs

### Status bar

- `lbl_wifi` — WiFi icon, green when connected
- `lbl_title` — "Vejret i Holmeenegen"
- `lbl_time` — current time HH:MM

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
| Backlight | GPIO2 switch (`id: backlight`), `restore_mode: ALWAYS_ON` |
| On touch | Resumes LVGL if paused; turns backlight on if off |

## Phase 2 plan

- [x] ESP32-S3 hardware config — verified with real board config
- [ ] Flash to hardware and confirm display + touch work
- [ ] WiFi + HA API integration (meteobridgesql integration)
- [ ] Populate card content areas with real sensor data
- [ ] Replace `lbl_time` dummy text with ESPHome time component
