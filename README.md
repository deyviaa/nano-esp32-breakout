# 🛠️ ArdNanoESP32 DevBoard

> *A compact, feature-packed carrier board for the Arduino Nano ESP32 — bridging the gap between bare breadboard chaos and a polished development kit.*

---

## 📖 Overview

This project is a custom-designed **PCB carrier board** built around the **Arduino Nano ESP32** (dual-core Xtensa LX7, Wi-Fi + BLE). Sized to an approximate **iPhone footprint**, the board is designed as an intermediate prototyping platform with a philosophy of **hardware-level safety** ("student-proofing") — all peripherals are wired with protection resistors, clean logic states, and deliberate design choices baked in at the schematic level.

The board seats the Nano ESP32 in ZIF-style socket headers and exposes every GPIO to parallel jumper pin headers for external breadboard connectivity, while also integrating a full suite of on-board peripherals ready for immediate use.

**Current Status:**

```
✅ Schematic design — Complete & verified
✅ Component footprints — All assigned & resolved
✅ Git version control — Initialized (cross-platform)
🔄 PCB Layout & Routing — In Progress
⬜ Gerber Generation & Fab — Pending
⬜ Assembly & Bring-up — Pending
```

---

## ⚡ Power Architecture

The board runs two accessible power rails sourced from the Nano ESP32's onboard regulators:

| Rail | Source | Purpose |
|------|--------|---------|
| **5V** | `VUSB` | WS2812B LED, raw power header |
| **3.3V** | Nano onboard LDO | All logic peripherals, GPIO I/O, I2C bus |
| **GND** | Common ground | Shared reference for all rails |

All power rails are broken out to dedicated 4-pin headers on the board edge for external use.

---

## 🧩 Peripheral Map

```
┌─────────────────────────────────────────────────────────┐
│               Arduino Nano ESP32 Carrier Board           │
│                                                         │
│  ┌──────────────┐     ┌─────────────┐                   │
│  │ Nano ESP32   │     │  4x LEDs    │ ── 1kΩ CLR ── GPIO │
│  │  (socketed)  │     │  4x Buttons │ ── 10kΩ PD ── GPIO │
│  └──────────────┘     │  4x Switches│ ── 1kΩ SER ── GPIO │
│                        └─────────────┘                   │
│  ┌──────────────┐     ┌─────────────┐                   │
│  │  WS2812B     │     │  Qwiic/     │                   │
│  │  RGB LED     │     │  STEMMA QT  │ ── SDA(A4)/SCL(A5) │
│  │  (5V + 470Ω) │     │  JST-SH 4P  │ ── 4.7kΩ pull-ups  │
│  └──────────────┘     └─────────────┘                   │
│  ┌──────────────┐     ┌─────────────┐                   │
│  │  Alps RK09K  │     │  12mm Piezo │                   │
│  │  Rotary Pot  │     │  Buzzer     │ ── SW mute ── GPIO  │
│  │  (VDiv/A0)   │     │  (220Ω buf) │                   │
│  └──────────────┘     └─────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

---

## 🔌 Peripheral Specifications

### 💡 LEDs × 4
- Generic standard LEDs, **active-high**
- **1kΩ** current-limiting resistors in series
- Clean GPIO drive, no floating states

### 🔘 Push Buttons × 4
- Momentary tactile switches
- **10kΩ pull-down** resistors → guaranteed `LOW` at rest
- No debounce hardware (handle in firmware or use `INPUT_PULLUP`)

### 🎚️ Slide Switches × 4 (Stateful GPIO Inputs)
- **SS12D00G-G3** SPDT Slide Switches (2.54mm pitch)
- Hard-wired: Pin 1 → 3.3V, Pin 3 → GND, Pin 2 → MCU
- **1kΩ series resistor** between common and GPIO — prevents a dead short if pin is accidentally configured as `OUTPUT LOW`
- ⚠️ *Avoid routing these to ESP32 strapping pins (GPIO0, GPIO3, GPIO45, GPIO46)*

### 🌈 WS2812B Addressable RGB LED
- **5050** form factor, driven from **VUSB (5V)** rail
- **100nF decoupling cap** placed physically adjacent to VDD/GND
- Data driven by 3.3V GPIO through **470Ω series resistor** (signal protection + level tolerance)
- `DOUT` left unconnected — ready for future LED chain expansion

### 🔁 Qwiic / STEMMA QT Connector (I2C)
- **JST-SH SM04B-SRSS-TB**, 1.0mm pitch, 4-pin
- Industry-standard pinout: `GND | 3V3 | SDA | SCL`
- Routed to hardware I2C: **SDA → A4**, **SCL → A5**
- **Two 4.7kΩ pull-up resistors** on SDA and SCL to 3.3V (open-drain compliant)
- Compatible with the entire Sparkfun/Adafruit Qwiic/STEMMA QT ecosystem

### 🎛️ Rotary Potentiometer
- **Alps RK09K** 9mm single-gang
- Wired as a safe voltage divider: Pin 1 → 3.3V, Pin 3 → GND, Wiper → Analog GPIO (A0–A7)
- Footprint includes 2 mechanical snap-in anchor pads for PCB stability

### 🔊 Piezo Buzzer
- **12mm PCB-mount** piezo, standard 7.62mm through-hole pitch
- **220Ω buffer resistor** on signal line
- **Hardware mute switch** (SS12D00 SPDT): one throw connects signal → buzzer positive; mute position leaves the throw floating (no drain-to-GND waste)

---

## 🗂️ Schematic Hierarchy

The project uses a hierarchical schematic structure in KiCad (4 sheets):

| Sheet | File | Contents |
|-------|------|----------|
| `/` | `ArdNanoESP32_DevBoard.kicad_sch` | Top-level hierarchy |
| `/MCU_Pins_Sockets/` | `MCU_Pins_Sockets.kicad_sch` | Nano ESP32 socket + jumper headers |
| `/PowerRails/` | `PowerRails.kicad_sch` | VUSB, 3V3, GND breakout headers |
| `/Peripherals/` | `Peripherals.kicad_sch` | All on-board peripherals |

---

## 🔧 KiCad Footprint Notes & Resolutions

Several non-obvious footprint decisions were made to match real-world, off-the-shelf components:

| Component | Footprint Assigned | Reason |
|---|---|---|
| SS12D00G-G3 Switches | `PinHeader_1x03_P2.54mm_Vertical` | Amazon-sourced units have **2.54mm pitch** (not standard 2mm) |
| Slide Switch Symbol | Generic `SW_SPDT` | Replaced Wuerth proprietary symbol — Pin 2 must be common to prevent 3V3↔GND short |
| JST-SH I2C Connector | `JST_SH_SM04B-SRSS-TB_1x04-1MP_P1.00mm_Horizontal` | Standard Qwiic/STEMMA 1.0mm pitch |
| Piezo Buzzer | `Buzzer_12x9.5mm_CR2032_Vertical` | Matches 7.62mm THT pin spacing |
| Alps Potentiometer | `Potentiometer_Alps_RK09K_Single_Vertical` | 3 signal pins + 2 mechanical anchor pads |
| Resistors (THT) | `R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` | Matches physical inventory |
| Resistors (SMD) | `R_0603_1608Metric_Pad1.05x0.95mm_HandSolder` | Hand-solder pad relief |

---

## 🌿 Git Workflow & Version Control

The repo is structured for **cross-platform KiCad development** (Windows PC ↔ MacBook Air).

### `.gitignore` covers:
```
# KiCad auto-generated / user-specific files
*.kicad_prl        # Per-user UI layout state
fp-info-cache      # Local footprint cache (rebuilt automatically)
*-bak              # KiCad backup files
*.lck              # Lock files (open-file indicators)
*_autosave-*       # Crash recovery autosaves
```

### ⚠️ Critical Workflow Rule

KiCad `.kicad_sch` and `.kicad_pcb` files are **binary-adjacent** — they do not merge cleanly. Follow this loop **strictly**:

```
┌─────────────────────────────────────────────┐
│                                             │
│   1. git pull  ← ALWAYS before opening     │
│   2. Open KiCad & do work                  │
│   3. Close KiCad (flushes file writes)     │
│   4. git add / commit / push               │
│                                             │
│   ❌ Never leave files open across machines │
│   ❌ Never commit while KiCad is open       │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 🧰 Tools & Software

| Tool | Version | Purpose |
|------|---------|---------|
| **KiCad** | 8.0.x | Schematic capture & PCB layout |
| **Git** | Any | Version control |
| **Arduino IDE / PlatformIO** | Latest | Firmware development |

---

## 📐 Design Constraints

- **Target Form Factor:** ~iPhone footprint (≈ 71mm × 150mm)
- **MCU:** Arduino Nano ESP32 (seated, not soldered)
- **Logic Level:** 3.3V throughout
- **Power Input:** USB via Nano ESP32 onboard USB-C

---

## 🚧 Roadmap

- [x] Schematic capture (all 4 sheets)
- [x] ERC pass — no errors
- [x] Footprint assignment & verification
- [x] Git repo initialization + `.gitignore`
- [ ] PCB layout — component placement
- [ ] PCB routing — all nets
- [ ] DRC pass — no errors
- [ ] 3D model review
- [ ] Gerber generation
- [ ] PCB fabrication (JLCPCB / PCBWay)
- [ ] Board bring-up & hardware testing
- [ ] Firmware examples / demo sketches

---

## 📄 License

*To be determined.*

---

> *Designed in KiCad. Iterated on breadboards. Built to survive students.*
