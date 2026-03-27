# ESPHome Current Monitor

ESPHome configuration for the small ESP32-C3 OLED board with a built-in 0.42 inch display. The node reads three Home Assistant current sensors, shows them on the onboard OLED, and drives a 12-pixel WS2812 strip as a simple 3-column current meter.

## Hardware

- Board: ESP32-C3 OLED Development Board 0.42 inch
- MCU: ESP32-C3
- Display controller: SSD1306
- Display size: 72x40 active pixels
- Built-in OLED wiring:
  - `SCL -> GPIO6`
  - `SDA -> GPIO5`
- LED strip wiring:
  - `WS2812 data -> GPIO2`
  - `12 LEDs`, arranged as `3 columns x 4 rows`

Important board note:

- The reseller PDF shows exposed I2C pins on `GPIO9/GPIO8`.
- The onboard OLED is not on those exposed pins.
- The OLED is wired internally to `GPIO6/GPIO5`.

## Home Assistant Entities

The config imports these Home Assistant sensors:

- `sensor.shelly_pro_3em_1_phase_a_current`
- `sensor.shelly_pro_3em_1_phase_b_current`
- `sensor.shelly_pro_3em_1_phase_c_current`

They are displayed as:

- `L1`
- `L2`
- `L3`

## OLED Behavior

- Refresh interval: `500ms`
- Label format: `L1`, `L2`, `L3`
- Value format: `xx.xxA`
- Unavailable format: `--.--A`
- Alert threshold: `16.0A`
- If a value is above `16.0A`, the numeric value blinks continuously
- The current repo configuration is set up for upside-down mounting with:
  - `rotation: 0`
  - `flip_x: false`
  - `flip_y: false`

## LED Strip Behavior

The repo also contains a WS2812 current meter:

- Entity name: `Current LEDs`
- 3 columns:
  - column 1 = `L1`
  - column 2 = `L2`
  - column 3 = `L3`
- 4 rows per column:
  - row 1 = green
  - row 2 = yellow
  - row 3 = orange
  - row 4 = red

Level mapping:

- `<= 0A`: 0 LEDs
- `0A .. 4A`: 1 LED
- `4A .. 8A`: 2 LEDs
- `8A .. 12A`: 3 LEDs
- `12A .. 16A`: 4 LEDs
- `> 16A`: top red LED blinks

Each row color is exposed as a separate light profile so the colors can be adjusted from Home Assistant:

- `LED Row 1 Color Profile`
- `LED Row 2 Color Profile`
- `LED Row 3 Color Profile`
- `LED Row 4 Color Profile`

## Repo Contents

- [`esp32-c3-oled-currents.yaml`](./esp32-c3-oled-currents.yaml): main ESPHome configuration
- [`ESP32-C3 OLED Development Board 0.42 Inch OLED WiFi Bluetooth Module Ceramic Antenna Wifi Bluetooth ESP32 C3 Development Board.pdf`](./ESP32-C3%20OLED%20Development%20Board%200.42%20Inch%20OLED%20WiFi%20Bluetooth%20Module%20Ceramic%20Antenna%20Wifi%20Bluetooth%20ESP32%20C3%20Development%20Board.pdf): original board PDF used for initial pinout reference

## Secrets

The YAML expects these secrets:

- `wifi_ssid`
- `wifi_password`
- `ota_password`
- `api_key`
- `ap_password`

`api_key` must be a valid ESPHome API encryption key, not a placeholder string.

## Flashing And OTA Notes

This repo intentionally does not hardcode a static IP.

If OTA via the ESPHome dashboard falls back to `current-monitor.local` and mDNS resolution fails, use the CLI with the device IP directly:

```bash
esphome run esp32-c3-oled-currents.yaml --device <device-ip>
```

or:

```bash
esphome upload esp32-c3-oled-currents.yaml --device <device-ip>
```

The config name on the network is:

- `current-monitor`

## Board / Framework Settings

- Board: `airm2m_core_esp32c3`
- Framework: Arduino
- Logger UART disabled with `baud_rate: 0`
- API encryption enabled

## Notes

- This config started as an OLED-only current display and was extended with the LED strip current meter.
- The OLED font and layout are tuned for the 72x40 display size.
