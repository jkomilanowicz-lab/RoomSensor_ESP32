# Azure Forest RoomSensor ESP32

**ESPHome firmware for the Azure Forest ESP32 HA Board V1.0**

A custom PCB integrating an AHT20 temperature/humidity sensor and LD2410C 24GHz mmWave human presence radar into Home Assistant, with Bluetooth proxy capability.

---

## Hardware

| Component | Details |
|-----------|---------|
| MCU | ESP32-D0WD-V3 (rev3, 4MB flash) |
| Temp/Humidity | AHT20 (I2C) |
| Presence Radar | HLK-LD2410C 24GHz mmWave (UART + GPIO OUT) |
| USB Bridge | CP2102n |

### Pin Mapping

| Signal | ESP32 GPIO | Notes |
|--------|-----------|-------|
| AHT20 SDA | GPIO21 | I2C SDA |
| AHT20 SCL | GPIO22 | I2C SCL |
| LD2410C OUT | GPIO4 | Binary presence output |
| LD2410C RX ← ESP TX | GPIO17 | UART TX2 |
| LD2410C TX → ESP RX | GPIO16 | UART RX2 |
| Onboard LED | GPIO2 | Controllable via HA |

---

## Features

- **Temperature & Humidity** — AHT20 with 30s update interval, -7°C self-heating offset
- **Human Presence Radar** — LD2410C moving/still detection, distance and energy sensors
- **Bluetooth Proxy** — ESP32 acts as a BT/BLE proxy for Home Assistant
- **LED Control** — Toggle onboard LED from HA
- **Restart Button** — Reboot device from HA
- **Device Info** — Designer, board, uptime, WiFi signal, IP, MAC all exposed in HA
- **Easy Provisioning** — Improv Serial + Improv BLE for first-time setup without manual config
- **OTA Updates** — Flash over WiFi after initial USB flash

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

### First-Time Setup via BLE (No USB needed after initial flash)
On first boot with no WiFi configured, open Home Assistant → **Settings → Devices & Services** — the device will appear for adoption. HA will guide you through WiFi provisioning via Bluetooth.

---

## Known Hardware Notes

- **AHT20 sold as AHT10**: Use `variant: AHT20` in ESPHome config — do not use AHT10 variant
- **SCL is GPIO22**: Not GPIO23. Verify against your board silkscreen
- **LD2410C UART labels are from LD2410C's perspective**: PCB "TX" = LD2410C transmits = ESP RX (GPIO16)
- **I2C frequency**: 50kHz required for reliable AHT20 operation (default 400kHz causes issues)
- **Boot delay**: 2s on_boot delay required for AHT20 initialization

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
