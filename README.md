# Azure Forest RoomSensor ESP32

**ESPHome firmware for the Azure Forest ESP32 HA Board V1.0**

A custom PCB integrating an AHT20 temperature/humidity sensor and LD2410C 24GHz mmWave human presence radar into Home Assistant.

> **Note:** This firmware was designed for the Azure Forest ESP32 HA Board V1.0 custom PCB, but it will work equally well on a standard ESP32 development board wired on a breadboard. No code changes are needed — just wire the components to the same GPIO pins listed below.

---

## Hardware

### Custom PCB (Azure Forest ESP32 HA Board V1.0)
A purpose-built PCB with the ESP32, AHT20, and LD2410C integrated in a compact form factor.

### Breadboard Alternative
No PCB? No problem. Any standard **ESP32 development board** (e.g. ESP32 DevKit V1) will work. You'll need:

| Component | Where to get |
|-----------|-------------|
| ESP32 DevKit (any 38-pin or 30-pin board) | Amazon, AliExpress, etc. |
| AHT20 breakout module | AliExpress, Adafruit, etc. |
| HLK-LD2410C presence radar module | AliExpress, HLK direct |
| Jumper wires + breadboard | Anywhere |

Wire everything to the GPIO pins listed below and the firmware runs identically.

---

## Pin Mapping

| Signal | ESP32 GPIO | Notes |
|--------|-----------|-------|
| AHT20 SDA | GPIO21 | I2C SDA |
| AHT20 SCL | GPIO22 | I2C SCL |
| LD2410C OUT | GPIO4 | Binary presence output |
| LD2410C RX ← ESP TX | GPIO17 | UART TX2 |
| LD2410C TX → ESP RX | GPIO16 | UART RX2 |
| Blue LED | GPIO2 | Onboard LED, controllable via HA |

> **LD2410C wiring note:** The PCB labels on the LD2410C module (TX/RX) are from the *module's* perspective, which is opposite to the ESP32's. Connect LD2410C **TX** → ESP **GPIO16** (RX) and LD2410C **RX** → ESP **GPIO17** (TX).

---

## Features

- **Temperature & Humidity** — AHT20 with 30s update interval; temperature and humidity offsets adjustable from HA (persist to flash)
- **Human Presence Radar** — LD2410C moving/still detection, distance and energy sensors
- **LD2410C Calibration** — All radar tuning controls exposed in HA: per-gate move/still thresholds (g0–g8), max distance gates, detection timeout, distance resolution, baud rate, OUT pin level, engineering mode
- **LED Status Indicator** — Solid on while connecting, 3 pulses when WiFi connects, then off
- **LED Control** — Toggle blue LED manually from HA
- **Restart Button** — Reboot device from HA
- **Device Info** — Designer, board, uptime, WiFi signal, IP, MAC all exposed in HA
- **Easy Provisioning** — Improv Serial for first-time USB setup
- **OTA Updates** — Flash over WiFi after initial USB flash
- **Auto Update Notifications** — Device checks GitHub for new firmware every 6h, shows update in HA

---

## Setup

### Requirements
- [ESPHome](https://esphome.io) (tested on v2025.2.2)
- Home Assistant with ESPHome integration

### First Flash (USB)
1. Clone this repo
2. Create a `secrets.yaml` in the project folder (see below)
3. Flash via USB:
```bash
esphome run roomsensor_esp32.yaml --device /dev/ttyUSB0
```
On macOS the device typically appears as `/dev/cu.SLAB_USBtoUART`

### secrets.yaml
Create this file locally — **never commit it**:
```yaml
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
ap_password: "fallback-ap-password"
api_encryption_key: "your-base64-32-byte-key"
ota_password: "your-ota-password"
```

Generate an API encryption key:
```bash
python3 -c "import secrets, base64; print(base64.b64encode(secrets.token_bytes(32)).decode())"
```

### Subsequent Flashes (OTA)
```bash
esphome run roomsensor_esp32.yaml --device azureforest-roomsensor-1.local
```

---

## Calibration

### Temperature & Humidity Offsets
The AHT20 sensor sits close to the ESP32 which generates heat. A temperature offset is applied to compensate. Both offsets are adjustable directly from Home Assistant (no reflash needed) and persist to flash:

- **Temperature Offset** — default `-7.0°C`. Tune with a reference thermometer.
- **Humidity Offset** — default `0%`. Tune with a reference hygrometer.

### LD2410C Radar Calibration
Full per-gate calibration is exposed in HA under the device's config entities:

- **Max Move/Still Distance Gate** — limits detection range (gates × 0.75m each)
- **Detection Timeout** — how long after last detection before clearing presence
- **Gate 0–8 Move/Still Threshold** — sensitivity per zone; lower = more sensitive
- **Engineering Mode** — enables live per-gate energy readout for fine-tuning

---

## Known Hardware Notes

- **AHT20 sold as AHT10**: Many AHT20 modules are sold and labelled as AHT10. Use `variant: AHT20` in the config regardless.
- **SCL is GPIO22**: Not GPIO23. Verify against your board or breadboard wiring.
- **LD2410C UART labels**: PCB labels are from LD2410C's perspective — see wiring note above.
- **I2C frequency**: 50kHz required for reliable AHT20 operation (default 400kHz causes issues on some modules).
- **Boot delay**: 2s on_boot delay required for AHT20 initialization.

---

## Flashing Multiple Boards

All boards use identical hardware. Edit `name:` and `friendly_name:` in the YAML before flashing each board:

```yaml
esphome:
  name: azureforest-roomsensor-2
  friendly_name: AzureForest RoomSensor 2
```

---

## Designed By

**Jack Omilanowicz** — [Azure Forest](https://github.com/jkomilanowicz-lab)

---

## License

MIT License — free to use, modify, and distribute.

*Firmware development assisted by Claude AI (Anthropic).*
