# Arduino Nano ESP32 Carrier Board: Project Summary

## 1. Project Overview & Direction

The project is a custom printed circuit board (PCB) acting as a compact, feature-rich carrier/development board for the **Arduino Nano ESP32**. Designed to fit an approximate "iPhone footprint," it serves as an intermediate testing and prototyping platform bridging the gap between a bare breadboard and a full commercial development kit. The board exposes the Nano's GPIO to jumper headers while integrating a suite of robust, on-board peripherals for immediate physical computing, status indication, and sensor integration.

**Current State:** The schematic design is complete, logically verified, and fully annotated. All component footprints have been assigned, resolving pitch and symbol mismatches. The project is currently transitioning to the PCB layout and routing phase. Git version control has been initialized for cross-platform development.

---

## 2. Core Architecture & Power

* **Microcontroller:** Arduino Nano ESP32 (Dual-core, WiFi/Bluetooth capable).
* **Mounting:** Socket headers to seat the Nano ESP32, accompanied by parallel pin headers for standard jumper wire connectivity to external breadboards.
* **Power Rails:** On-board 5V (VUSB) and 3.3V rails accessible for peripheral and external development use.

---

## 3. Peripheral Specifications & Schematic Logic

Throughout the design phase, specific focus was placed on hardware-level safety ("student-proofing") and adherence to communication standards.

### Standard I/O

* **LEDs (x4):** Standard generic LEDs wired active-high with 1kΩ current-limiting resistors.
* **Push Buttons (x4):** Momentary tactile switches wired with standard 10kΩ pull-down resistors to ensure a clean `LOW` state when unpressed.

### Stateful Inputs (Toggle Switches)

* **Component:** SS12D00G-G3 (SPDT Slide Switches).
* **Logic:** Wired as state selectors without pull resistors (Pin 1 to 3.3V, Pin 3 to GND, Pin 2/Common to MCU). This provides a hard `HIGH` or `LOW` state.
* **Safety Integration:** A 1kΩ resistor is placed in series between the switch common and the ESP32 GPIO. This prevents a dead short (and blown pin) if the GPIO is accidentally programmed as an `OUTPUT LOW` while the switch is tied to 3.3V.
* *Note: Care must be taken to avoid utilizing the ESP32's strapping pins for these hard-wired switches.*

### I2C Expansion (Qwiic / STEMMA QT)

* **Component:** 4-pin JST-SH connector (1.0mm pitch).
* **Standard:** Follows the strict industry pinout: Pin 1 (GND), Pin 2 (3.3V), Pin 3 (SDA), Pin 4 (SCL).
* **Routing:** Explicitly mapped to the Arduino Nano ESP32 hardware I2C pins: **SDA to A4** and **SCL to A5**.
* **Logic:** Includes two 4.7kΩ pull-up resistors connecting SDA and SCL to the 3.3V rail to satisfy open-drain requirements.

### Analog Input

* **Component:** Alps RK09K 9mm Rotary Potentiometer.
* **Logic:** Wired as a safe voltage divider. Pin 1 to 3.3V, Pin 3 to GND, and Pin 2 (Wiper) routed to an Analog-capable pin (A0-A7).

### Addressable RGB Status LED

* **Component:** WS2812B (5050 form factor).
* **Power:** Fed directly from the 5V (VUSB) rail for stability, utilizing a 100nF decoupling capacitor placed physically close to the VDD/GND pins.
* **Data Line:** Driven by a 3.3V GPIO through a 470Ω series protection resistor. The DOUT pin is left unconnected, allowing for potential future chaining.

### Audio Debugging

* **Component:** 12mm Piezo Buzzer (PCB mount).
* **Hardware Mute:** An SS12D00 SPDT slide switch is placed in series with the buzzer signal.
* **Logic:** The switch connects the signal (buffered by a 220Ω resistor) to the buzzer's positive terminal in one position. In the mute position, the throw is left completely floating/unconnected to prevent unnecessary current drain to GND.

---

## 4. Footprint Assignments & CAD (KiCad) Resolutions

Several critical footprint and symbol discrepancies were caught and resolved prior to generating the netlist:

* **Switch Symbol Correction:** Replaced proprietary Wuerth switch symbols with generic `SW_SPDT` symbols in KiCad. This ensured that Pin 2 accurately represented the common pivot, preventing a catastrophic dead short between 3.3V and GND.
* **Switch Footprint Hack:** Assigned `PinHeader_1x03_P2.54mm_Vertical` to the slide switches to accommodate the 2.54mm pitch of the Amazon-sourced SS12D00G-G3 switches, ensuring a perfect fit without bent pins.
* **I2C Connector:** Assigned `JST_SH_SM04B-SRSS-TB_1x04-1MP_P1.00mm_Horizontal` (or Vertical).
* **Buzzer:** Assigned `Buzzer_12x9.5mm_CR2032_Vertical` to account for standard 7.62mm pitch through-hole mounting.
* **Potentiometer:** Assigned `Potentiometer_Alps_RK09K_Single_Vertical`, allowing for the 3 mono pins and 2 mechanical snap-in anchors.
* **Resistors:** Standardized based on physical inventory (THT `R_Axial_DIN0207...` vs. SMD `0603_HandSolder`).

---

## 5. Version Control Infrastructure

A Git repository has been initialized to allow seamless development between a primary PC and a MacBook Air.

* **Ignore Protocol:** A strict `.gitignore` is in place, specifically excluding KiCad's temporary files, backups (`*-bak`), lock files (`*.lck`), user-specific UI settings (`*.kicad_prl`), and the local footprint cache (`fp-info-cache`).
* **Workflow:** Development relies on a strict pull-before-opening, commit-after-closing loop to prevent merge conflicts in the binary-like CAD files.
