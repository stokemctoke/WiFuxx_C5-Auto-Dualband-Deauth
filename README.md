# 🔥 WiFuxx v1.1 — ESP32-C5 Autonomous Dual-Band Deauther

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: ESP32-C5](https://img.shields.io/badge/Platform-ESP32--C5-blue)](https://www.espressif.com/en/products/socs/esp32-c5)
[![Status: Production Ready](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)]()

**WiFuxx** is a compact, autonomous deauthentication tool designed for the XIAO ESP32-C5. It scans for nearby Wi-Fi networks, filters by signal strength, and launches targeted deauth attacks — all displayed on a tiny OLED screen.

---

## ⚠️ Legal Disclaimer

> **IMPORTANT:** Laws regarding Wi-Fi deauthentication vary significantly by country. In many jurisdictions, using a deauther against networks you do not own or have explicit permission to test is **illegal** and may result in criminal charges, fines, or imprisonment.
>
> **This tool is intended for:**
> - ✅ Testing your own network security
> - ✅ Educational purposes in controlled environments
> - ✅ Authorised penetration testing
>
> **DO NOT USE on public, neighbour, or any networks without written permission.**
>
> **By using this software, you accept full responsibility for your actions.**

---

## 📡 Features

- **⚡ Fully Autonomous** — No web interface, no control needed. Power on and it works.
- **📊 Smart Targeting** — Only attacks APs with signal > -70 dBm (strongest nearby networks).
- **🖥️ OLED Display** — Real-time status on a 128×64 screen:
  - Number of detected APs
  - Current status (IDLE / SCAN / ATTACK)
  - Scrolling list of target SSIDs
- **🔫 Dual-Band Support** — Attacks both 2.4 GHz and 5 GHz networks.
- **🚀 High Performance** — Optimised channel-hopping attack pattern.
- **📝 Serial Logging** — Detailed logs via USB serial for debugging.
- **🔋 Battery Ready** — Can run on battery via XIAO battery pads.

---

## 🛠️ Hardware Required

| Component | Quantity | Notes |
|-----------|----------|-------|
| XIAO ESP32-C5 | 1 | Main controller |
| 0.96" OLED Display (I2C) | 1 | 128×64, SSD1306 driver |
| Breadboard & Jumper Wires | — | For prototyping |
| USB-C Cable | 1 | Power and programming |

---

## 🔌 Wiring Diagram

Connect the OLED display to the XIAO ESP32-C5 as follows:

```
XIAO ESP32-C5          OLED Display
┌─────────────┐        ┌──────────┐
│             │        │          │
│     3V3     ├───────►│   VCC    │
│             │        │          │
│     GND     ├───────►│   GND    │
│             │        │          │
│  D4 (SDA)   ├───────►│   SDA    │  (GPIO23)
│             │        │          │
│  D5 (SCL)   ├───────►│   SCL    │  (GPIO24)
│             │        │          │
└─────────────┘        └──────────┘
```

**Pin Mapping Reference:**

| XIAO Pin | GPIO | OLED Connection |
|----------|------|-----------------|
| D4 (SDA) | GPIO23 | SDA (Data) |
| D5 (SCL) | GPIO24 | SCL (Clock) |
| 3V3 | — | VCC (Power) |
| GND | — | Ground |

> **Note:** The OLED must be a 3.3V compatible model. Most 0.96" I2C OLEDs work perfectly at 3.3V.

---

## 📥 Installation

### Prerequisites

- [ESP-IDF v5.0 or later](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/)
- XIAO ESP32-C5 board support
- USB-C cable

### Step 1: Clone the Repository

```bash
git clone https://github.com/yourusername/wifuxx.git
cd wifuxx
```

### Step 2: Configure the Project

```bash
idf.py set-target esp32c5
idf.py menuconfig
```

In `menuconfig`, verify:

- **Component config → Wi-Fi → WiFi enable** is checked
- **Component config → I2C → I2C enable** is checked

### Step 3: Build and Flash

```bash
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

> Replace `/dev/ttyUSB0` with your actual serial port.

---

## 🎮 Operation

### Automatic Mode

WiFuxx runs a continuous autonomous loop from the moment it powers on:

```
IDLE (30s) → SCAN (5–10s) → ATTACK (300s) → IDLE → (repeat)
```

**Display status during each phase:**

| Phase | Display |
|-------|---------|
| IDLE | Waiting for next scan cycle |
| SCAN | Scanning for networks, counting strong APs |
| ATTACK | Actively deauthenticating targets (shows packet counts) |

### OLED Display Layout

```
┌────────────────────────────────┐
│ WiFuxx v1.1                    │  ← Title
│ APs: 7                         │  ← Number of strong APs found
│ Status: ATTACK                 │  ← Current state
│ NETGEAR_123                    │  ← Target 1 (scrolls if >5)
│ BT-Hub_456                     │  ← Target 2
│ Starlink_789                   │  ← Target 3
│ TP-Link_XXX                    │  ← Target 4
│ Xfinity_YYY                    │  ← Target 5
└────────────────────────────────┘
```

### Configuration Parameters

Edit these values at the top of `main.c` to customise behaviour:

```c
#define BAD_SIGNAL_THRESHOLD       -70    // Attack APs stronger than this (dBm)
#define MAX_TARGETS                10     // Maximum APs to target
#define AUTO_SCAN_INTERVAL_SEC     30     // Seconds between scans
#define AUTO_ATTACK_DURATION_SEC   300    // Attack duration in seconds
```

---

## 📊 Serial Monitor Output

Connect via serial monitor to see detailed logs:

```bash
idf.py monitor
```

Example output:

```
I (1234) WiFuxx: 🔍 Scanning for networks...
I (5678) WiFuxx: ✅ Found 15 total networks
I (5679) WiFuxx: 🎯 Targeting APs with signal > -70 dBm:
I (5680) WiFuxx:   [0] NETGEAR_123 (CH: 6, 2.4GHz, RSSI: -45)
I (5681) WiFuxx:   [1] BT-Hub_456  (CH: 1, 2.4GHz, RSSI: -52)
I (5682) WiFuxx: 💥 MULTI-TARGET ATTACK STARTED!
I (5683) WiFuxx: 💥 [5/300 sec] Total: 12450 pkt | PPS: 2490 | Targets: 7
```

---

## 🔧 Troubleshooting

| Problem | Solution |
|---------|----------|
| OLED display blank | Check wiring, especially SDA/SCL. Run an I2C scanner sketch to verify address. |
| No deauth effect | Verify promiscuous mode is enabled — check serial for `"Failed to enable promiscuous mode"`. |
| Device not scanning | Ensure Wi-Fi is initialised correctly — serial logs will show errors. |
| Compilation errors | Run `idf.py clean` and rebuild. Ensure ESP-IDF is v5.0+. |

---

## 🧪 Testing

### Safe Mode (No RF Activity)

To test the OLED and boot sequence without any RF activity:

```c
#define AUTO_MODE_ENABLED 0   // In main.c
```

This initialises all hardware but never scans or attacks — useful for verifying your wiring.

---

## 📈 Performance

| Metric | Value |
|--------|-------|
| Packet rate | ~2,500 packets/second (aggregate across all targets) |
| Attack latency | < 1 ms between channel switches |
| Display update rate | 1 Hz (negligible CPU impact) |
| Memory usage | ~50 KB heap, ~500 KB flash |

---

## 🔮 Future Ideas

- Battery level monitoring (XIAO battery pads)
- Battery icon on OLED
- External antenna mod
- 3D printed enclosure

---

## 🙏 Credits

- Original concept by Stokes
- ESP-IDF framework by [Espressif](https://www.espressif.com/)
- SSD1306 driver adapted from public domain sources

---

## 📜 Licence

MIT Licence — see the [LICENSE](LICENSE) file for details.

---

## ⚡ Quick Start

1. Wire OLED: `VCC→3V3`, `GND→GND`, `SDA→D4`, `SCL→D5`
2. Flash with `idf.py flash monitor`
3. Watch the OLED show AP count and status
4. Device attacks automatically every 30 seconds

> **Remember:** Only use on your own networks! 🛡️

---

*WiFuxx — Because sometimes you need to fuxx about (with your own network 😉)*
