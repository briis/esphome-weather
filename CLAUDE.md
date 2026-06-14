# ESPHome Weather Display — Project Reference

## What this is

800×480 LVGL v9 weather dashboard on ESP32-S3 using ESPHome. Danish UI ("Vejret i Holmeenegen").
Design: 3-column dark card grid on pure black background.

## Current phase

**Phase 2 — Temperature, Wind, and Sun & UV cards populated.** Grid layout verified working on Waveshare ESP32-S3-Touch-LCD-7. SDL simulator used for visual iteration; hardware target flashed via USB and confirmed. Next: remaining cards (Forecast, Rain, Barometer).

## Files

| File | Purpose |
| --- | --- |
| `weather-common.yaml` | Shared: fonts, images, LVGL layout, `lvgl_update_clock` script. Included by both targets. |
| `esphome-weather-sdl.yaml` | SDL simulator target (`host` platform). Layout iteration + mock data scripts. |
| `esphome-weather.yaml` | ESP32-S3 hardware target. Waveshare 7" 800×480. HA sensors. |
| `fonts/materialdesignicons-webfont.ttf` | MDI icon font. |
| `images/wind_bearing_indicator.png` | Current wind bearing needle (24×24). |
| `images/wind_bearing_avg_indicator.png` | Average bearing needle (18×18). |

## How to run

SDL simulator (layout iteration):

```sh
esphome run esphome-weather-sdl.yaml
```

Prerequisites: `brew install sdl2` + `pip install esphome`

Flash to hardware:

```sh
esphome run esphome-weather.yaml
```

## Layout structure

12×12 internal grid (each cell ~64×39 px). Widgets span 4 columns × 5 rows for main cards.

```text
┌──────────────────────────────────────────────────────────────────┐
│  STATUS BAR  (wifi icon left · title center · time right)        │
├───────────────────┬───────────────────┬──────────────────────────┤
│  Temperature      │    Wind           │     Sun & UV             │
│  col 0–3, row 1–5 │    col 4–7, row 1–5│    col 8–11, row 1–5  │
├───────────────────┼───────────────────┼──────────────────────────┤
│  (placeholder)    │  (placeholder)    │     (placeholder)        │
│  col 0–3, row 6   │  col 4–7, row 6   │    col 8–11, row 6      │
└───────────────────┴───────────────────┴──────────────────────────┘
```

## Color palette

Colors are defined as `substitutions:` at the top of `weather-common.yaml`. Change the hex value there to retheme globally; individual target files can override any key in their own `substitutions:` block.

| Substitution ID | Hex | Role |
| --- | --- | --- |
| `clr_bg` | `0x000000` | Background |
| `clr_card` | `0x1C2130` | Card background |
| `clr_separator` | `0x2A3348` | Card separator line |
| `clr_tick` | `0xE1E1E1` | Wind meter tick colour |
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
| `font_sm` | Roboto | 14 | Default / card titles |
| `font_md` | Roboto | 18 | Mid-size values |
| `font_lg` | Roboto | 22 | Status bar |
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

- `wifi_label` — WiFi strength icon (MDI) + colour driven by RSSI. Initial state: wifi-off red. SDL simulates random strength every minute; hardware uses `wifi_signal` sensor (60 s) + `on_connect`/`on_disconnect` for immediate feedback.
- `display_heder` — "Vejret i Holmeengen" (static title)
- `time_label` — HH:MM, updated via `lvgl_update_clock` script. SDL: triggered by `on_time` every minute. Hardware: immediate on `on_time_sync`, then every minute via `on_time`.

### Temperature card (col 0–3)

- `lbl_temp_high` — daily high (font_md, with ↑ arrow icon to left)
- `circle_temp` — 180×180 meter, transparent fill, 6 px coloured border driven by temperature threshold
- `lbl_temp_current` — current temperature (font_xxl, centered inside circle)
- `lbl_temp_low` — daily low (font_md, with ↓ arrow icon to left)

HA sensors (entity IDs set via substitutions, overridden in `esphome-weather.yaml`):

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_temp_current` | `sensor.unknown_temperature` | Current temp + circle colour |
| `ha_temp_high` | `max_temperature_today` | Daily high (attribute of current sensor) |
| `ha_temp_low` | `min_temperature_today` | Daily low (attribute of current sensor) |

### Wind card (col 4–7)

- `circle_wind_border` — 180×180 meter, 4 px blue border, 360° scale with 17 tick marks
- `bearing_indicator` — current bearing needle image (24×24 px, orbits outer edge). Rotation set via `lv_get_needle_angle_for_value` + `lv_obj_set_style_transform_rotation`.
- `bearing_avg_indicator` — 10-min average bearing needle image (18×18 px, same orbit radius)
- `lbl_wind_cardinal` — cardinal direction of current bearing (e.g. "NØ"), top-centre of meter
- `lbl_wind_speed` — current wind speed m/s (font_xxl, centred)
- `lbl_wind_gust` — gust speed m/s (font_md, bottom area)

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_wind_bearing` | `sensor.unknown_wind_bearing` | Current bearing → needle + cardinal |
| `ha_wind_speed` | `sensor.unknown_wind_speed` | Wind speed display |
| `ha_wind_gust` | `sensor.unknown_wind_gust` | Gust speed display |
| `ha_wind_bearing_avg` | `sensor.unknown_wind_bearing_avg` | 10-min avg bearing needle |

#### Bearing indicator pivot — schema workaround

ESPHome's LVGL schema rejects negative `pivot_x` values even though they are valid C++. The YAML uses `pivot_x: 0` as a placeholder; an `on_boot` lambda fixes all three values:

```cpp
// bearing_indicator (24×24 image, orbit radius 78 px)
lv_obj_set_style_x(id(bearing_indicator), 78, LV_PART_MAIN);          // orbit radius
lv_obj_set_style_transform_pivot_x(id(bearing_indicator), -66, LV_PART_MAIN);  // image_w/2 - R = 12 - 78
lv_obj_set_style_transform_pivot_y(id(bearing_indicator), 12, LV_PART_MAIN);   // image_h/2

// bearing_avg_indicator (18×18 image, orbit radius 78 px)
lv_obj_set_style_x(id(bearing_avg_indicator), 78, LV_PART_MAIN);
lv_obj_set_style_transform_pivot_x(id(bearing_avg_indicator), -69, LV_PART_MAIN);  // 9 - 78
lv_obj_set_style_transform_pivot_y(id(bearing_avg_indicator), 9, LV_PART_MAIN);
```

Formula: `x_offset = R`, `pivot_x = image_width/2 - R`, `pivot_y = image_height/2`. This places the rotation pivot exactly at the scale centre so the needle orbits at radius R.

### Sun & UV card (col 8–11)

- `circle_uv` — 180×180 meter, border colour driven by UV index (WHO scale)
- `lbl_uv` — UV index integer value (font_xxl, centred inside circle)
- `lbl_uv_unit` — static unit label
- `circle_solar` — solar radiation meter
- `lbl_solar` — solar radiation value W/m² (font_xxl)
- `lbl_solar_unit` — static unit label

UV border colour scale (WHO):

| UV index | Level | Colour |
| --- | --- | --- |
| 0–2 | Low | green (`clr_wifi_good`) |
| 3–5 | Moderate | yellow (`clr_temp_cool`) |
| 6–7 | High | orange (`clr_temp_warm`) |
| 8–10 | Very High | red (`clr_temp_hot`) |
| 11+ | Extreme | violet `0x9C27B0` |

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_uv_index` | `sensor.unknown_uv_index` | UV index → label + circle colour |
| `ha_solar_radiation` | `sensor.unknown_solar_radiation` | Solar radiation → label |

## Hardware config (`esphome-weather.yaml`)

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
| OTA | Turns backlight off on begin |
| Captive portal | Enabled |
| Time component id | `system_time` (matches id used in `lvgl_update_clock` script in common) |

### Package strategy

`esphome-weather.yaml` (hardware) pulls `weather-common.yaml` from the **remote Forgejo repo** at `https://git.digitalhomes.dk/briis/esphome-weather` (ref: `main`, refresh: 10 s, auth via `!secret forgejo_token`).

`esphome-weather-sdl.yaml` (SDL simulator) uses a local `!include weather-common.yaml`.

Pushing an update to `weather-common.yaml` on `main` propagates to the device on next compile/OTA without reflashing.

## Known gotchas

- **`::time(nullptr)` required in lambdas** — the `homeassistant` time component introduces the `esphome::time` namespace, making a bare `time()` call ambiguous. Always use `::time(nullptr)` to reference the C stdlib function.
- **`lv_obj_set_style_text_color` selector** — use `0` (not `LV_PART_MAIN`) for the selector argument in lambdas to avoid header-include issues.
- **`scrollbar_mode: "OFF"` needs quotes** — YAML parses bare `OFF` as boolean `false`.
- **ESPHome rejects negative `pivot_x`** — the YAML schema validator requires `pivot_x >= 0`. Use `0` as a placeholder and fix with `lv_obj_set_style_transform_pivot_x` in `on_boot`. Must also fix `x` offset and `pivot_y` — all three are linked (see Bearing indicator section above).
- **`lv_image_set_pivot` is wrong for meter indicators** — use `lv_obj_set_style_transform_pivot_x/y` instead. The indicator object is an `lv_obj_t*`; `lv_image_set_pivot` targets the image zoom pivot, not the rotation transform pivot.
- **HA sensor substitutions belong in the hardware target** — defined as placeholders in `weather-common.yaml` but overridden in `esphome-weather.yaml`. Main-file substitutions take precedence over package substitutions in ESPHome.
- **`esphome-weather.yaml` must be kept in sync in two places** — local dev copy AND the HA ESPHome addon config folder. The common file only needs to be on Forgejo.
- **Time component id must be `system_time`** — `weather-common.yaml`'s `lvgl_update_clock` script references `id(system_time)`. Both SDL (host time) and hardware (HA time) must use this id.

## Phase 2 plan

- [x] ESP32-S3 hardware config — verified with real board config
- [x] Flash to hardware — display and touch confirmed working
- [x] Live clock (`time_label`) — updates every minute, correct on both SDL and hardware
- [x] WiFi signal icon (`wifi_label`) — RSSI-based icon + colour, simulated on SDL
- [x] Named color substitutions — all hex values replaced with `${clr_*}` references in common file
- [x] Temperature card — circle outline + current temp + daily high/low, live HA sensors
- [x] Wind card — compass meter with current + avg bearing needles, speed, gust, cardinal direction
- [x] Sun & UV card — UV meter with WHO-scale colour border + solar radiation display, live HA sensors
- [ ] Forecast card
- [ ] Rain card
- [ ] Barometer card
