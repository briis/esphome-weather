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
| Sun & UV | UV index with WHO-scale colour border, solar radiation W/m² |
| Barometer | Half-circle pressure gauge (960–1060 hPa), trend symbol and description |
| Feels like / Dewpoint | Apparent temperature and dew point |
| Wind averages | 10-min average wind speed and daily max gust |
| Rain | Circle gauge with today's total, yesterday's total, and current rain rate |
| Forecast | Weather condition image, high/low temps, precipitation probability + amount, condition description |
| AQI | EPA-scale colour circle with AQI index, PM2.5 and PM10 particulate matter |

## Files

| File | Purpose |
| --- | --- |
| `weather-common.yaml` | Shared layout, fonts, widgets — pulled by the hardware target from Forgejo |
| `weatherdisplay.yaml` | ESP32-S3 hardware target with HA sensor bindings |
| `weatherdisplay-sdl.yaml` | SDL simulator target for local layout iteration |

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

## Home Assistant integration

Sensor entity IDs are configured via substitutions in `weatherdisplay.yaml`. The hardware target fetches `weather-common.yaml` directly from the Forgejo repository at compile time, so updating the shared layout propagates to the device on the next OTA without reflashing.
