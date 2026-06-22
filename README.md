# ESPHome Weather Display

An ESPHome-based weather dashboard running on an ESP32-S3 with a 7" 800×480 touchscreen. The UI is built with LVGL v9 and displays live sensor data pulled from Home Assistant.

## Hardware

- **Board**: Waveshare ESP32-S3-Touch-LCD-7 (16 MB flash, 8 MB PSRAM)
- **Display**: 800×480 RGB panel, MIPI interface
- **Touch**: GT911 capacitive touchscreen

## Dashboard

Dark-themed 3-column card grid with the following widgets:

| Card | Data shown |
| --- | --- |
| Temperature | Current temp with colour-coded circle (cold/cool/warm/hot), daily high/low |
| Wind | Compass meter with current + 10-min average bearing needles, speed, gust, cardinal direction |
| Sun & UV | UV index with WHO-scale colour border, solar radiation W/m², today's sunrise and sunset times |
| Barometer | Half-circle pressure gauge (960–1060 hPa), trend symbol and description |
| Feels like / Dewpoint / Humidity | Apparent temperature, dew point, and relative humidity |
| Wind averages | 10-min average wind speed and daily max gust |
| Rain | Circle gauge with today's total, yesterday's total, and current rain rate |
| Forecast | Weather condition image, high/low temps, precipitation probability + amount, condition description |
| AQI | EPA-scale colour circle with AQI index, PM2.5 and PM10 particulate matter |

## Files

| File | Purpose |
| --- | --- |
| `weather-common.yaml` | Shared layout, fonts, widgets — pulled from Forgejo by the hardware target |
| `weatherdisplay.yaml` | Hardware run file — thin wrapper pointing to Forgejo packages |
| `weatherdisplay-config-hw.yaml` | Full hardware config: board, sensors, night mode, substitutions — lives on Forgejo |
| `weatherdisplay-sdl.yaml` | SDL run file — thin wrapper for local simulator |
| `weatherdisplay-config-sdl.yaml` | SDL simulator config: mock data scripts, host platform |

## Running

**SDL simulator** (no hardware needed):

```sh
esphome run weatherdisplay-sdl.yaml
```

Prerequisites: `brew install sdl2` and `pip install esphome`

**Flash to hardware**:

```sh
esphome run weatherdisplay.yaml
```

## Deployment

`weatherdisplay.yaml` is a thin wrapper that pulls `weatherdisplay-config-hw.yaml` and `weather-common.yaml` from the Forgejo repository at compile time. The normal workflow for config or layout changes is to **push to Forgejo `main`** — the device picks up the update on the next OTA triggered from the HA ESPHome add-on, without needing a local flash.

## Home Assistant integration

Sensor entity IDs are configured via substitutions in `weatherdisplay-config-hw.yaml`.

The night mode feature exposes a boolean switch ("Nat mode") and four number entities (off hour/minute, on hour/minute) directly in HA. When enabled, the display turns off and on on the configured schedule. Touching the display during night mode wakes it for 2 minutes before it goes back to sleep.
