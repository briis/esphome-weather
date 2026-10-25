# ESPHome Weather Display — Project Reference

## What this is

800×480 LVGL v9 weather dashboard on ESP32-S3 using ESPHome. Danish UI ("Vejret i Holmeenegen").
Design: 3-column dark card grid on pure black background.

## Current phase

**Phase 3 complete.** All cards populated with live HA sensors. Extended with humidity display, sunrise/sunset times, configurable night mode, and split file structure for GitHub-driven deployment. Verified on Waveshare ESP32-S3-Touch-LCD-7.

## Files

| File | Purpose |
| --- | --- |
| `weather-common.yaml` | Shared: fonts, images, LVGL layout, `lvgl_update_clock` script. Pulled from GitHub by hardware; included locally by SDL. |
| `weatherdisplay.yaml` | Hardware run file — thin wrapper. Declares `packages: remote_package_files:` pointing to GitHub. Rarely needs editing. |
| `weatherdisplay-sdl.yaml` | SDL run file — thin wrapper. Includes `weatherdisplay-config-sdl.yaml` and `weather-common.yaml` locally. |
| `weatherdisplay-config-hw.yaml` | Full hardware config: ESP32-S3 board, PSRAM sdkconfig, WiFi, HA sensors, night mode, substitutions. Lives on GitHub. |
| `weatherdisplay-config-sdl.yaml` | SDL simulator config: host platform, SDL display, mock data scripts. Local only. |
| `fonts/materialdesignicons-webfont.ttf` | MDI icon font. |
| `images/wind_bearing_indicator.png` | Current wind bearing needle (24×24). |
| `images/wind_bearing_avg_indicator.png` | Average bearing needle (18×18). |

## How to run

SDL simulator (layout iteration):

```sh
esphome run weatherdisplay-sdl.yaml
```

Prerequisites: `brew install sdl2` + `pip install esphome`

Flash to hardware (initial flash or after changing board-level config):

```sh
esphome run weatherdisplay.yaml
```

**Normal workflow for layout / sensor changes:** push `weather-common.yaml` and/or `weatherdisplay-config-hw.yaml` to GitHub `main`. The device picks up changes on next OTA from the HA ESPHome add-on — no local flash needed.

## Deployment — package strategy

`weatherdisplay.yaml` is a minimal pointer:

```yaml
packages:
  remote_package_files:
    url: https://github.com/briis/esphome-weather
    files:
      - weatherdisplay-config-hw.yaml
      - weather-common.yaml
    ref: main
    username: briis
    password: !secret esphome_weather_token
    refresh: 10s
```

Both config files are pulled from GitHub at compile time. Pushing a change to either file on `main` propagates to the device on the next OTA without touching the ESPHome add-on UI.

`weatherdisplay-sdl.yaml` uses a local `!include` for both files — no GitHub dependency.

**Note on HA add-on cache:** The add-on can serve a stale copy even with `refresh: 10s`. If the device shows an older layout, compile and flash locally: `esphome run weatherdisplay.yaml --device /dev/cu.usbmodemXXXX`.

## Layout structure

12×12 internal grid (each cell ~64×39 px).

```text
┌──────────────────────────────────────────────────────────────────┐
│  STATUS BAR  (wifi icon left · title center · time right)   row 0│
├───────────────────┬───────────────────┬──────────────────────────┤
│  Temperature      │    Wind           │     Sun & UV             │
│  col 0–3, row 1–5 │  col 4–7, row 1–5 │   col 8–11, row 1–4    │
├───────────────────┼───────────────────├──────────────────────────┤
│  Feels like /     │  Wind avg /       │     Barometer            │
│  Dewpoint /       │  Max gust         │   col 8–11, row 5–7      │
│  Humidity         │  col 4–7, row 6–7 │                          │
│  col 0–3, row 6–7 │                   │                          │
├───────────────────┼───────────────────┼──────────────────────────┤
│  Rain             │  Forecast         │     AQI                  │
│  col 0–3, row 8–11│  col 4–7, row 8–11│   col 8–11, row 8–11   │
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
| `font_xxs` | Roboto | 8 | Smallest labels (rain widget sub-values) |
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
| `F058C` | weather-pouring | Rain card header icon |
| `F17FE` | sun-wireless | Solar radiation |
| `F0510` | thermometer-lines | Feels like / apparent temp |
| `F029A` | gauge | Barometer |
| `F1594` | trending-up | Pressure rising |
| `F1595` | trending-down | Pressure falling |
| `F1598` | trending-neutral | Pressure stable |
| `F0595` | weather-forecast | Forecast card header icon |
| `F0D7C` | air-filter | AQI card header icon |

## Widget IDs

### Status bar

- `wifi_label` — WiFi strength icon (MDI) + colour driven by RSSI. Initial state: wifi-off red. SDL simulates random strength every minute; hardware uses `wifi_signal` sensor (60 s) + `on_connect`/`on_disconnect` for immediate feedback.
- `display_heder` — "Vejret i Holmeengen" (static title)
- `time_label` — HH:MM, updated via `lvgl_update_clock` script. SDL: triggered by `on_time` every minute. Hardware: immediate on `on_time_sync`, then every minute via `on_time`.

### Temperature card (col 0–3, row 1–5)

- `lbl_temp_high` — daily high (font_md, with ↑ arrow icon to left)
- `circle_temp` — 180×180 meter, transparent fill, 6 px coloured border driven by temperature threshold
- `lbl_temp_current` — current temperature (font_xxl, centered inside circle)
- `lbl_temp_low` — daily low (font_md, with ↓ arrow icon to left)

HA sensors (entity IDs set via substitutions, overridden in `weatherdisplay-config-hw.yaml`):

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_temp_current` | `sensor.unknown_temperature` | Current temp + circle colour |
| `ha_temp_high` | `max_temperature_today` | Daily high (attribute of current sensor) |
| `ha_temp_low` | `min_temperature_today` | Daily low (attribute of current sensor) |

### Wind card (col 4–7, row 1–5)

- `circle_wind_border` — 180×180 meter, 4 px blue border, 360° scale with 17 tick marks
- `bearing_indicator` — current bearing needle image (24×24 px, orbits outer edge). Rotation set via `lv_get_needle_angle_for_value` + `lv_obj_set_style_transform_rotation`.
- `bearing_avg_indicator` — 10-min average bearing needle image (18×18 px, same orbit radius)
- `lbl_wind_cardinal` — 16-point cardinal direction of current bearing (e.g. "NNØ"), top-centre of meter. Formula: `d[(int)((b + 11.25f) / 22.5f) % 16]`
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

### Sun & UV card (col 8–11, row 1–4)

- `circle_uv` — 115×115 meter, border colour driven by UV index (WHO scale)
- `lbl_uv` — UV index integer value (font_xl, centred inside circle)
- `lbl_uv_unit` — static "UV Index" label
- `circle_solar` — 115×115 solar radiation meter
- `lbl_solar` — solar radiation value W/m² (font_xl)
- `lbl_solar_unit` — static "W/m²" label
- `lbl_sunrise` — today's sunrise time (font_xs, dim), updated by `ts_sunrise` text sensor
- `lbl_sunset` — today's sunset time (font_xs, dim), updated by `ts_sunset` text sensor

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
| `ha_sunrise` | `sensor.unknown_sunrise` | Sunrise time string → `lbl_sunrise` |
| `ha_sunset` | `sensor.unknown_sunset` | Sunset time string → `lbl_sunset` |

**Sunrise/sunset must be `text_sensor:`** — HA exposes these as string entities. Declaring them under `sensor:` causes a `LoadProhibited` crash (see Known gotchas). Hardware IDs are `ts_sunrise` / `ts_sunset`.

### Barometer card (col 8–11, row 5–7)

- `baro_gauge` — 140×140 half-circle arc (180°–0°, range 960–1060 hPa), teal indicator arc
- `lbl_pressure` — current pressure value (font_xl, centred inside arc)
- `lbl_pressure_unit` — static "hPa" label (font_xs, dim)
- `pressure_trend_symbol` — MDI trend arrow icon (mdi_sm), colour driven by trend
- `pressure_trend_text` — trend description text (e.g. "Stabil"), font_sm

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_pressure` | `sensor.unknown_pressure` | Current pressure → arc value + label |
| `ha_pressure_trend` | `sensor.unknown_pressure_trend` | Trend direction (unused, reserved) |
| `ha_pressure_trend_value` | `sensor.unknown_pressure_trend_value` | Trend value → symbol icon + colour + text |

### Feels like / Dewpoint / Humidity sub-card (col 0–3, row 6–7)

Three-column grid (`grid_columns: [fr(1), fr(1), fr(1)]`):

- `temp_feels_like` — apparent temperature (font_lg)
- `temp_dewpoint` — dew point temperature (font_lg)
- `lbl_humidity` — relative humidity % (font_lg), formatted `"%.0f%%"`

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_feels_like` | `sensor.unknown_apparent_temperature` | Apparent temperature |
| `ha_dew_point` | `sensor.unknown_dewpoint` | Dew point |
| `ha_humidity` | `sensor.unknown_humidity` | Relative humidity → `lbl_humidity` |

### Wind averages sub-card (col 4–7, row 6–7)

- `wind_10min_avg` — 10-minute average wind speed (font_lg)
- `wind_max_gust` — daily maximum gust (font_lg)

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_wind_speed_avg` | `sensor.unknown_wind_speed` | 10-min avg speed |
| `ha_wind_gust_high` | `sensor.unknown_wind_gust` | Daily max gust |

### Rain card (col 0–3, row 8–11)

- `circle_rain` — 140×140 meter, 6 px blue border
- `lbl_rain_yesterday` — yesterday's total mm (font_xs, dim, TOP_MID y:5 inside circle), formatted "I går: x,x"
- `rain_today` — today's total mm (font_xl, centred inside circle)
- Static "mm" label (font_sm, dim, CENTER y:20 inside circle)
- `lbl_rain_rate` — current rain rate (font_xs, dim, BOTTOM_MID y:-5 inside circle), formatted "x,x mm/h"

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_rain_today` | `sensor.unknown_rain_today` | Today's accumulated rain → `rain_today` label |
| `ha_rain_rate` | `sensor.unknown_rain_rate` | Current rain rate → `lbl_rain_rate` |
| `ha_rain_yesterday` | `sensor.unknown_rain_yesterday` | Yesterday's total → `lbl_rain_yesterday` |

### Forecast card (col 4–7, row 8–11)

- `fimg_clear_day` … `fimg_windy` — 11 weather condition images (64×64), only one visible at a time, toggled by condition code
- `lbl_fcst_high` — forecast high temperature (font_md, with ↑ arrow icon)
- `lbl_fcst_low` — forecast low temperature (font_md, with ↓ arrow icon, dim)
- `lbl_fcst_precip_prob` — precipitation probability % (font_md, blue, right-aligned)
- `lbl_fcst_precipitation` — precipitation amount mm (font_md, dim, right-aligned)
- `lbl_fcst_condition` — short condition text (font_sm, truncated with DOT)
- `lbl_fcst_description` — longer description (font_xs, dim, truncated with DOT)

HA sensors — all four numeric forecast sensors have `filters: - filter_out: nan` to prevent PSRAM crash on boot (see Known gotchas):

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_fcast_condition` | `sensor.unknown_fcast_condition` | Condition code → image switch |
| `ha_fcast_condition_txt` | `sensor.unknown_fcast_condition_txt` | Raw HA weather condition state (e.g. `partlycloudy`) → translated to Danish on-device → `lbl_fcst_condition` |
| `ha_fcast_description` | `sensor.unknown_fcast_description` | Long description → `lbl_fcst_description` |
| `ha_fcst_temp_high` | `sensor.unknown_fcst_temp_high` | Forecast high → `lbl_fcst_high` |
| `ha_fcst_temp_low` | `sensor.unknown_fcst_temp_low` | Forecast low → `lbl_fcst_low` |
| `ha_fcst_precip_prob` | `sensor.unknown_fcst_precip_prob` | Precipitation probability → `lbl_fcst_precip_prob` |
| `ha_fcst_precipitation` | `sensor.unknown_fcst_precipitation` | Precipitation amount → `lbl_fcst_precipitation` |

### AQI card (col 8–11, row 8–11)

- `circle_aqi` — 140×140 meter, 6 px border, colour driven by AQI level (EPA scale)
- `lbl_aqi` — AQI integer value (font_xl, centred inside circle)
- Static "AQI" label (font_sm, dim, BOTTOM_MID y:−15 inside circle)
- `lbl_pm25` — PM2.5 particulate matter (font_sm, BOTTOM_LEFT), formatted "x,x μg/m³"
- `lbl_pm10` — PM10 particulate matter (font_sm, BOTTOM_RIGHT), formatted "x,x μg/m³"

AQI border colour scale (EPA):

| AQI range | Level | Colour |
| --- | --- | --- |
| 0–50 | Good | green (`clr_wifi_good`) |
| 51–100 | Moderate | yellow (`clr_temp_cool`) |
| 101–150 | Unhealthy for Sensitive Groups | orange (`clr_temp_warm`) |
| 151–200 | Unhealthy | red (`clr_temp_hot`) |
| 201–300 | Very Unhealthy | violet `0x9C27B0` |
| 301+ | Hazardous | maroon `0x7E0023` |

HA sensors:

| Substitution | Default in common | Purpose |
| --- | --- | --- |
| `ha_aqi_index` | `sensor.unknown_aqi_index` | AQI value → `lbl_aqi` + circle colour |
| `ha_aqi_pm25` | `sensor.unknown_aqi_pm25` | PM2.5 → `lbl_pm25` |
| `ha_aqi_pm10` | `sensor.unknown_aqi_pm10` | PM10 → `lbl_pm10` |

## Night mode

Configurable display off/on schedule, exposed to HA as a switch and four number entities. All defined in `weatherdisplay-config-hw.yaml`.

### HA-visible entities

| Entity | Type | Default | Description |
| --- | --- | --- | --- |
| `night_mode_switch` | `switch` (template) | off | "Nat mode" — enable/disable the schedule. Turning off restores the backlight immediately. |
| `night_off_hour` | `number` | 22 | Hour to turn display off |
| `night_off_minute` | `number` | 0 | Minute to turn display off |
| `night_on_hour` | `number` | 7 | Hour to turn display back on |
| `night_on_minute` | `number` | 0 | Minute to turn display back on |

All number entities use `restore_value: true` so they survive reboots.

### Scripts

- `night_mode_check` (mode: single) — called every minute by `on_time` and `on_time_sync`. Compares current time against the off/on window using integer minute arithmetic. Supports midnight-spanning windows (e.g. 22:00–07:00). Skips if `night_wake_active` is true.
- `night_mode_sleep_timer` (mode: restart) — triggered by touch when night mode is on. Sets `night_wake_active = true`, turns backlight on, waits 2 minutes, clears the flag, then calls `night_mode_check` to re-evaluate.

### Global

- `night_wake_active` (bool, initial: false) — prevents `night_mode_check` from turning the display off during the 2-minute touch-wake window.

### Touch behaviour

When touched during night mode: `night_mode_sleep_timer` runs with `mode: restart`, so repeated touches reset the 2-minute countdown. When touched while night mode is disabled (or outside the night window): normal backlight-on logic.

## Hardware config (`weatherdisplay-config-hw.yaml`)

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
| On touch | Resumes LVGL if paused; then branches on `night_mode_switch` state (see Night mode) |
| OTA | Turns backlight off on begin |
| Captive portal | Enabled |
| Time component id | `system_time` (matches id used in `lvgl_update_clock` script in common) |

## Known gotchas

- **`::time(nullptr)` required in lambdas** — the `homeassistant` time component introduces the `esphome::time` namespace, making a bare `time()` call ambiguous. Always use `::time(nullptr)` to reference the C stdlib function.
- **`lv_obj_set_style_text_color` selector** — use `0` (not `LV_PART_MAIN`) for the selector argument in lambdas to avoid header-include issues.
- **`scrollbar_mode: "OFF"` needs quotes** — YAML parses bare `OFF` as boolean `false`.
- **ESPHome rejects negative `pivot_x`** — the YAML schema validator requires `pivot_x >= 0`. Use `0` as a placeholder and fix with `lv_obj_set_style_transform_pivot_x` in `on_boot`. Must also fix `x` offset and `pivot_y` — all three are linked (see Bearing indicator section above).
- **`lv_image_set_pivot` is wrong for meter indicators** — use `lv_obj_set_style_transform_pivot_x/y` instead. The indicator object is an `lv_obj_t*`; `lv_image_set_pivot` targets the image zoom pivot, not the rotation transform pivot.
- **HA sensor substitutions belong in the hardware target** — defined as placeholders in `weather-common.yaml` but overridden in `weatherdisplay-config-hw.yaml`. Main-file substitutions take precedence over package substitutions in ESPHome.
- **`weatherdisplay-config-hw.yaml` must stay in sync** — the copy on GitHub is what the device compiles from. If you edit locally and don't push, the next OTA will revert the change.
- **Time component id must be `system_time`** — `weather-common.yaml`'s `lvgl_update_clock` script references `id(system_time)`. Both SDL (host time) and hardware (HA time) must use this id.
- **String HA entities must be `text_sensor:`** — entities like `sensor.next_sunrise` deliver a string, not a float. Declaring them under `sensor:` (numeric platform) causes a `LoadProhibited` crash at runtime when the `on_value` lambda applies `%s` formatting to a float pointer. Use `text_sensor:` + `lv_label_set_text(id(lbl_foo), x.c_str())`.
- **NaN + `CONFIG_SPIRAM_RODATA` crash** — forecast sensors send NaN on first boot while the HA weather integration is initialising. With `CONFIG_SPIRAM_RODATA: "y"`, the ROM `_svfprintf_r` crashes when formatting NaN with `%.1f` (PSRAM cache access issue in ROM code). Fix: add `filters: - filter_out: nan` to every forecast sensor that uses `%.1f` in its `on_value` lambda.
- **Octal PSRAM boot-crash with ESP-IDF 5.5.4 / pioarduino 55.03.38-1** — newer ESP-IDF enables `CONFIG_ESPTOOLPY_FLASH_MODE_AUTO_DETECT` by default, which conflicts with OPI PSRAM init on the Waveshare board. Without PSRAM the LVGL framebuffer (1.5 MB) can't be allocated and the device boot-loops. Fix already in `weatherdisplay-config-hw.yaml` `sdkconfig_options`: `CONFIG_ESPTOOLPY_FLASH_MODE_AUTO_DETECT: ""`, `CONFIG_ESPTOOLPY_FLASHMODE_DIO: "y"`, `CONFIG_SPIRAM_FETCH_INSTRUCTIONS: "y"`, `CONFIG_SPIRAM_RODATA: "y"`.
- **HA add-on package cache can serve stale files** — `refresh: 10s` does not always invalidate the add-on's cached copy. Compile and flash locally to bypass the cache when needed.
