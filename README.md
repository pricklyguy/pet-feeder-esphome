# ESP32 Pet Feeder with Home Assistant Integration

An automated pet feeder built with ESP32 and ESPHome, featuring pulse-based portion control, manual feed button, and Star Wars-themed audio feedback.

## 🎵 Features

- **Pulse-based portion control** - Accurate dispensing using microswitch feedback
- **4 scheduled feeding times** - Fully configurable through Home Assistant
- **Manual feed button** - Physical button on the device for instant feeding
- **Star Wars audio feedback** - Short beep on start, Imperial March on completion
- **Daily portion tracking** - Automatic reset at midnight
- **Home Assistant integration** - Full automation and dashboard support
- **Web interface** - Built-in web server for device management

## 🛠️ Hardware Requirements

### Components
- ESP32 Dev Board
- Single relay module (3V) (can use 5V, just separate logic from coils)
- Microswitch (for portion counting)
- Passive buzzer
- 2N2222 NPN transistor
- Tactile push button (manual feed)
- 0.01µF (103) ceramic capacitor
- 1N4007 flyback diode
- 1kΩ resistor
- Pet feeder mechanism with motor

### Wiring Diagram

```
Motor Control:
ESP32 GPIO32 → Relay IN
Relay VCC → 3.3V
Relay GND → Shared Ground

Relay Side:
COM → 5V +
NO → Motor +
Motor – → Shared Ground

Flyback Diode (across motor terminals):
1N4007 Stripe (cathode) → Motor +
1N4007 Non-stripe (anode) → Motor – / GND

Pulse Counter (with debounce):
Microswitch → GPIO19
Microswitch → GND
0.01µF cap across microswitch terminals

Manual Button:
Button → GPIO25
Button → GND

Buzzer (with transistor driver):
GPIO13 → 1kΩ resistor → 2N2222 Base
2N2222 Collector → Buzzer (-)
2N2222 Emitter → GND
Buzzer (+) → 5V
```

## 📦 Installation

### 1. ESPHome Configuration

1. Copy `petfeeder.yaml` to your ESPHome config directory
2. Create a `secrets.yaml` file with your WiFi credentials:
   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   ```
3. Flash the ESP32:
   ```bash
   esphome run petfeeder.yaml
   ```

### 2. Home Assistant Package

1. Enable packages in your `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copy `pet_feeder.yaml` to `/config/packages/`

3. Restart Home Assistant

4. Configure your feeding schedule in Settings → Devices & Services → Helpers

## 🎮 Usage

### Scheduled Feeding

1. Go to Settings → Devices & Services → Helpers
2. Find the "Pet Feeder Slot X" helpers
3. Enable the slot, set time and portion count
4. The feeder will automatically dispense at the scheduled time

### Manual Feeding

**Physical Button:**
- Press the button on the device to dispense the currently set portion amount

**Home Assistant:**
- Use the manual feed scripts (1, 2, 3, or 5 portions)
- Or press the "Execute Feed Action" button with a custom portion count

### Portion Tracking

- Daily portions are tracked automatically
- View total portions dispensed in Home Assistant
- Counter resets at midnight

## ⚙️ Configuration

### Adjusting Portion Sizes

In `petfeeder.yaml`, change the max value:
```yaml
number:
  - platform: template
    name: "HA Target Pulses"
    id: ha_target_pulses
    min_value: 0
    max_value: 10  # Adjust this value
```

### Changing Audio Feedback

#### Short Beep (on start):
```yaml
- rtttl.play: "beep:d=4,o=5,b=100:16c6"
```

#### Star Wars Theme (on complete):
```yaml
- rtttl.play: "StarWars:d=8,o=6,b=180:f5,f5,f5,2a#5.,2f.,d#,d,c,2a#.,4f.,d#,d,c,2a#.,4f.,d#,d,d#,2c"
```

Replace with any [RTTTL string](http://esphome.io/components/rtttl.html) you prefer.

### Fine-tuning Pulse Counting

If portions are consistently off by 1:

**Adjust debounce time:**
```yaml
sensor:
  - platform: pulse_counter
    filters:
      - debounce: 900ms  # Increase or decrease
```

**Or adjust counting edge:**
```yaml
count_mode:
  rising_edge: INCREMENT  # Try swapping
  falling_edge: DISABLE   # these values
```

## 🔧 Troubleshooting

### Motor runs continuously
- Check GPIO pin assignment (avoid strapping pins like GPIO2, GPIO15)
- Verify relay is active-LOW or active-HIGH
- Add pull-down resistor if needed

### Pulse counter not working
- Verify microswitch wiring and continuity
- Check capacitor value (too large = missed pulses, too small = noise)
- Adjust debounce timing
- Try different GPIO pin

### Portions inconsistent (±1)
- Normal variation due to debounce timing
- Fine-tune debounce value (700-1000ms range)
- Ensure motor speed is consistent

### Device won't boot
- Remove status LED on GPIO2 (strapping pin)
- Check for shorts in wiring
- Verify power supply is adequate (500mA+)

### Buzzer clicks instead of beeping
- Verify buzzer is passive (requires PWM), not active
- Check transistor wiring (E-B-C pinout)
- Confirm LEDC output is configured correctly

## 📊 Home Assistant Dashboard

Multiple dashboard examples are provided in [`home-assistant/dashboard-examples.yaml`](home-assistant/dashboard-examples.yaml):

### Example 1: Full Mobile Dashboard (Neon Theme)
Beautiful mobile-optimized interface with:
- Glowing neon effects on enabled slots
- Color-coded manual feed buttons
- Responsive layout
- Touch-optimized controls

**Requirements:** HACS components `mushroom` and `card-mod`

### Example 2: Simple Schedule Card
Basic entities card showing just the feeding schedule.

### Example 3: Quick Feed Buttons
Compact button grid for manual feeding.

### Example 4: Statistics Card
Device status and feeding statistics.

See the [dashboard-examples.yaml](home-assistant/dashboard-examples.yaml) file for complete code and customization tips.

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

## 📝 License

This project is open source and available under the MIT License.

## 🙏 Acknowledgments

- ESPHome team for the amazing framework
- Home Assistant community
- And may the Force be with your pets! 🐾⭐

---

**Note:** This feeder mechanism should always be monitored initially to ensure proper operation. Never leave pets unattended with automated feeders until you're confident in their reliability.
