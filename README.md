# 🐾 PGC Pet Feeder – ESPHome

**Prickly Guy Creations** | Automated pet feeder built on ESPHome with full Home Assistant integration and standalone web UI.

Two production hardware versions, both sharing the same core firmware logic: pulse-based portion counting via microswitch, 4 configurable schedule slots stored on-device, 22 completion tunes, and a styled web interface that works without Home Assistant.

---

## Hardware Versions

### V2 – ESP32 DevKit
Custom PCB using an ESP32-DEVKIT-V1.  Single LED, passive buzzer, IRLZ44N MOSFET motor drive, optional OLED/button header.

**Schematic:** `schematics/Pet_Feeder_ESP32_v2.png`
**ESPHome config:** `v2-esp32dev/pgcpetfeeder.yaml`

| Pin | Function |
|-----|----------|
| GPIO32 | Motor (MOSFET gate) |
| GPIO19 | Microswitch (pulse counter) |
| GPIO25 | Status LED |
| GPIO23 | Buzzer (LEDC) |

### V3 – Seeed XIAO ESP32-C6
Custom PCB using the Seeed XIAO ESP32-C6. Two LEDs (green/red), passive buzzer, IRLZ44N motor drive, 4 buttons, OLED header, low-food sensor input.

**Schematic:** `schematics/PetFeederC6-Rev3.png`
**ESPHome config:** `v3-esp32c6/pgcpetfeeder-c6.yaml`

| Pin | Function |
|-----|----------|
| D0 | Motor (MOSFET gate) |
| D1 | Microswitch (pulse counter) |
| D2 | Buzzer (LEDC) |
| D3 | LED Green |
| D10 | LED Red |
| D6–D9 | Buttons 1–4 |
| D4/D5 | I2C SDA/SCL (OLED) |
| GPIO6 | Low food sensor |

---

## Firmware Features

- **Pulse-based portion counting** — microswitch interrupt with configurable lockout, no missed pulses
- **4 schedule slots** — hour/minute/portions stored in flash, survive reboots, no HA required
- **22 completion tunes** — selectable via HA or web UI (Adams Family → Star Wars and more)
- **Portions today + lifetime counter** — both persist across reboots
- **Standalone web UI** — custom CSS/JS served from GitHub Pages, works without HA
- **Full HA integration** — all controls exposed as native entities
- **Timeout protection** — if microswitch fails to count target pulses within 16s, motor stops and error tone plays

---

## Getting Started

### 1. Flash the ESP32

```bash
# Copy the correct yaml for your board to your ESPHome config directory
# Add to secrets.yaml:
#   wifi_ssid: "YourSSID"
#   wifi_password: "YourPassword"
#   ap_password: "fallback"

esphome run v2-esp32dev/pgcpetfeeder.yaml
# or
esphome run v3-esp32c6/pgcpetfeeder-c6.yaml
```

### 2. Rename for your device

Change the `substitutions` block at the top:

```yaml
substitutions:
  device_name: my_cat_feeder      # used for entity IDs
  friendly_name: My Cat Feeder    # used for display names
```

### 3. Home Assistant Dashboard

Copy the relevant dashboard YAML from `home-assistant/` and paste it into HA's dashboard editor (raw config mode).

- `dashboard-v2-esp32dev.yaml` — for V2 boards
- `dashboard-v3-c6.yaml` — for V3 boards

Replace the entity prefix (`rojas_feeder` or `twix_feeder`) with your `device_name`.

**Requirements:** [Mushroom](https://github.com/piitaya/lovelace-mushroom) and [card-mod](https://github.com/thomasloven/lovelace-card-mod) from HACS.

---

## Schedule Configuration

Schedules are stored entirely on the device — no HA automations or helpers needed.

Each slot has three settings:
- **Hour** (0–23)
- **Minute** (0–59)  
- **Portions** (1–10)

Plus an **Enabled** switch. Configure via the web UI at `http://<device-ip>` or through Home Assistant.

---

## Troubleshooting

**Motor runs but no pulses counted**
- Check microswitch wiring and continuity
- Adjust `pulse_lockout_ms` global (default 800ms) if pulses are being skipped

**Motor runs past target / doesn't stop**
- Verify microswitch is normally open (NO), not normally closed
- Try inverting the pin: `inverted: true`

**Timeout error tone plays every feed**
- Motor is too slow — increase timeout in `dispense` script (default 16s)
- Check power supply, motor may be undervoltage

**Device won't connect after flash**
- Connect to the fallback AP (`<device_name>-fallback`) and configure WiFi
- Check `secrets.yaml` has correct credentials

---

## Repository Structure

```
pet-feeder-esphome/
├── v2-esp32dev/
│   └── pgcpetfeeder.yaml          # V2 canonical template (ESP32 DevKit)
├── v3-esp32c6/
│   └── pgcpetfeeder-c6.yaml       # V3 canonical template (XIAO ESP32-C6)
├── schematics/
│   ├── Pet_Feeder_ESP32_v2.png    # V2 PCB schematic
│   └── PetFeederC6-Rev3.png       # V3 PCB schematic + board renders
├── home-assistant/
│   ├── dashboard-v2-esp32dev.yaml # HA dashboard for V2
│   └── dashboard-v3-c6.yaml      # HA dashboard for V3
└── README.md
```

---

## License

MIT — feel free to use, modify, and share. Credit appreciated but not required.

---

*Built by [Prickly Guy Creations](https://github.com/pricklyguy) 🌵*
