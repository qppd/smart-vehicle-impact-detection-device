# Bill of Materials (BOM)
## IoT-Based Vehicle Impact Detection System

---

### 2.1 Component List

| # | Component | Qty | Verified Specs | Notes | Product Link |
|---|-----------|-----|----------------|-------|--------------|
| 1 | ESP32 Development Board | 1 | Generic 38-pin variant, main MCU | **TBD: exact model/pinout** | [Lazada - ESP32 30/38-pin](https://www.lazada.com.ph/products/30-pins-and-38-pins-esp32-wifi-iot-development-board-i229344573-s21305417099.html) |
| 2 | MPU6050 Breakout | 1 | Soldered variant, I²C, accelerometer + gyroscope | Impact detection | [Lazada - MPU6050](https://www.lazada.com.ph/products/triple-axis-accelerometer-and-gyro-breakout-mpu6050-i309260737-s609286653.html) |
| 3 | SIM808 Module | 1 | GSM + GPS, TTL UART, DC044/V_IN/Li-Po inputs | Antennas included | [Lazada - SIM808](https://www.lazada.com.ph/products/pdp-i141829650-s161561560.html) |
| 4 | Grove LED — Green | 1 | Module form factor | Normal status | [Lazada - Grove LED](https://www.lazada.com.ph/products/grove-blue-red-green-purple-led-arduino-raspberry-pi-compatible-i1932813041-s8305193829.html) |
| 5 | Grove LED — Red | 1 | Module form factor | Emergency status | [Lazada - Grove LED](https://www.lazada.com.ph/products/grove-blue-red-green-purple-led-arduino-raspberry-pi-compatible-i1932813041-s8305193829.html) |
| 6 | Li-Po Battery 3.7V 2000 mAh | 2 | **PARALLEL / 1S2P** | Nominal 3.7V, Full 4.2V, ~4000 mAh | [Lazada - PKCELL Li-Po](https://www.lazada.com.ph/products/pkcell-lithium-ion-polymer-battery-37v-500mah-2000mah-dashcamwireless-keyboard-battery-i315312163-s17644383464.html) |
| 7 | TP4056 USB-C Charger | 1 | 5V input, ~1A charge, protection circuit | **Charger only, NOT 5V regulator** | [Lazada - TP4056](https://www.lazada.com.ph/products/type-c-micro-usb-5v-1a-18650-tp4056-lithium-battery-charger-module-charging-board-with-protection-i109875975-s12903019466.html) |
| 8 | Boost Converter | 1 | XL6009E1 / LM2577-type | Boost 3.7V → 5.0V for ESP32 VIN | [Lazada - Boost Converter](https://www.lazada.com.ph/products/pdp-i127879529-s137115635.html) |
| 9 | Buzzer | 1 | Active/passive TBD | Audible warning | [Lazada - Buzzer](https://www.lazada.com.ph/products/pdp-i3474748260-s17874906532.html) |
| 10 | Miniature Rocker Switch | 1 | ON-OFF, main power | Switch main positive supply | [Lazada - Rocker Switch](https://www.lazada.com.ph/products/pdp-i3015510213-s14819033850.html) |
| 11 | Tinned Copper Wire | 1 spool | Multiple AWG options | Power + signal wiring | [Lazada - Tinned Wire](https://www.lazada.com.ph/products/model-1007-18awg-20awg-22-awg-24awg-tinned-copper-wire-5-color-spool-i3154761672-s16587708998.html) |
| 12 | USB Type-C Cable | 1+ | Power + data variants | ESP32 programming, TP4056 charging | [Lazada - USB-C Cable](https://www.lazada.com.ph/products/usb-type-c-data-cable-i2990384044-s14655790370.html) |
| 13 | DT-830D Digital Multimeter | 1 | Yellow variant | Voltage, continuity, resistance | [Lazada - DT-830D](https://www.lazada.com.ph/products/dt-830b-dt-830d-digital-multimeter-lcd-acdc-7501000v-mini-portable-multimeter-for-ohm-voltmeter-aneng-620a-47-inch-large-lcd-screen-automatic-manual-intelligent-true-rms-digital-multimeter-i2292363253-s14093338657.html) |
| 14 | IP65 ABS Enclosure | 1 | 160 × 160 × 90 mm, transparent lid | Weatherproof housing | [Lazada - Enclosure](https://www.lazada.com.ph/products/weatherproof-enclosure-ip65-nema-4-abs-transparent-lid-i2906352389-s15191892279.html) |
| 15 | Dupont Wire Kit | 1 | Assorted | Prototyping only | [Lazada - Dupont Kit](https://www.lazada.com.ph/products/pdp-i2658568071-s12646800983.html) |
| 16 | TNT SIM Card | 1 | Registered | GSM/SMS testing | (User supplied) |
| 17 | Electrolytic Capacitor 1000µF 25V | 1 | Low ESR preferred, 25V min | SIM808 input decoupling (smooths 2A TX bursts) | [Lazada - 1000uF 25V](https://www.lazada.com.ph/products/pdp-i4246157653-s23691141314.html) |
| 18 | 4.7 kΩ Resistor | 0–2 | ¼W, any tolerance | I²C pull-ups — **only if your MPU6050 breakout has none** (most GY-521-style boards already do). See WIRING.md 6.2 | (Not needed if breakout has pull-ups) |
| 19 | Heat-shrink / Electrical Tape | 1 | — | Insulating the parallel pack joints (SETUP.md Step 1) | (Consumable) |
| 20 | HBK Travel Car Charger — **C103 USB-C variant** | 1 | 12/24V accessory socket → **5V output on USB Type-C**, dual USB output | **Charging source for the TP4056**: its Type-C output connects directly to the **TP4056 Type-C port**. **Order variant `C103 TYPE-C`** (not C102 / micro-V8 / iOS) | [Lazada - HBK Car Charger C103 USB-C](https://www.lazada.com.ph/products/hbk-travel-car-charger-c102-c103-dual-usb-output-for-andriod-micro-v8-and-type-c-ios-onhand-sale-i4112106562-s133615824176.html) |

**Not listed here:** enclosure mounting hardware (standoffs, screws, cable
glands, sealant, Velcro) is itemized separately in **SETUP.md §14.18**.
Antennas ship with the SIM808 module (item 3). Lazada product links above are
**UNVERIFIED** (live availability not checked) — substitute equivalent parts if
a link is dead.

---

### 2.2 Detailed Component Specifications

#### ESP32 Development Board (Generic 38-pin)
- **Architecture:** ESP32-D0WDQ6 (dual-core) or similar
- **Flash:** 4 MB typical
- **USB-to-UART:** CP2102 / CH340 / CH9102 — **TBD: verify actual chip**
- **I/O:** 3.3V logic
- **Pinout:** 38-pin — **TBD: map actual GPIO to pin numbers**
- **Power:** VIN 5V, 3.3V output, GND
- **Bootstrapping pins:** GPIO0, GPIO2, GPIO12, GPIO15 — avoid for peripherals
- **Strapping warning:** Do not pull GPIO0/2/12/15 high/low at boot

#### MPU6050 (Soldered Breakout)
- **Interface:** I²C (default address 0x68, AD0 low)
- **Accelerometer:** ±2g / ±4g / ±8g / ±16g (configurable)
- **Gyroscope:** ±250 / ±500 / ±1000 / ±2000 °/s
- **VDD:** 2.375V – 3.46V (typically 3.3V)
- **Logic:** 3.3V compatible
- **INT pin:** Available for interrupt-driven detection
- **Mounting:** 4-hole breakout, soldered headers

#### SIM808 Module
- **Power Inputs:**
  - DC044: 5–26V (barrel jack)
  - V_IN: 5–26V (pin header)
  - Li-Po interface: 3.5–4.2V (direct battery)
- **VBAT (module supply):** 3.4V–4.4V (SIMCom), **peak up to 2 A** during GSM TX
  bursts — this project powers it straight from the 1S pack, so the pack,
  wiring (18–20 AWG) and the 1000µF capacitor must handle that burst
- **GSM:** Quad-band 850/900/1800/1900 MHz
- **GPS:** L1 C/A, 22 tracking / 66 acquisition channels, −165 dBm tracking,
  −147 dBm cold start (SIMCom SIM808 specification)
- **TTFF:** typically ~30 s cold start, ~1 s hot start (manufacturer-typical —
  **not measured on this project: UNVERIFIED**)
- **Accuracy:** <2.5 m CEP (manufacturer spec, open-sky conditions)
- **UART:** TTL, RXD/TXD/VMCU
- **VMCU:** some SIM808 breakout boards expose a VMCU jumper/resistor that selects
  the UART logic level (commonly 3.3V / 5V). **SET TO 3.3V FOR ESP32 — ESP32
  GPIO inputs are NOT 5V tolerant.** *Verify your actual board: NEED USER INPUT.*
  (The bare SIM808 module runs ~2.8V IO; only the breakout's level selection
  matters here.)
- **Antenna:** GSM + GPS antennas included
- **Current:** Up to 2A peak during TX burst

#### Grove LEDs (Green & Red)
- **Interface:** 4-pin Grove (GND, VCC, NC, SIG)
- **Logic:** 3.3V / 5V compatible
- **Current:** ~10-20 mA per LED
- **Mounting:** Panel-mountable

#### Li-Po Battery (2× 3.7V 2000 mAh)
- **Configuration:** PARALLEL / 1S2P
- **Nominal voltage:** 3.7V (typical operating point)
- **Full charge voltage:** 4.2V (per cell)
- **Minimum safe discharge voltage:** 3.0V per cell (do not discharge below this)
- **Cut-off voltage (recommended):** 3.2V (protects cell longevity)
- **System operating limit (this design):** the SIM808 needs ≥3.4V and the
  XL6009E1 boost needs ≥3.6V input, so **recharge at ~3.7V pack voltage** —
  the electronics stop working before the cells reach their 3.0V safety floor
- **Total capacity:** ~4000 mAh (parallel connection)
- **⚠️ Safety:** Must verify voltage match (<0.1V diff) before paralleling
- **⚠️ Protection:** Each cell must have protection circuit or use protected cells
- **⚠️ Warning:** Do not exceed 4.2V per cell during charging; overcharge causes fire

#### TP4056 USB-C Charger/Protection
- **Input:** 5V USB-C, ~1A charge current
- **Output:** Battery terminals (B+, B-)
- **Protection:** Overcharge, over-discharge, over-current, short-circuit
- **⚠️ NOT a 5V regulator** — cannot power ESP32 directly
- **⚠️ 1S only** — designed for single-cell (3.7V nominal)
- **Implication for 1S2P:** Parallel cells appear as one larger 1S cell — OK if cells matched
- **Verify on board:** DW01 protection IC, FS8205 MOSFETs, charge current resistor

#### HBK Travel Car Charger (5V source for TP4056 charging)
- **Purpose:** provides the **5V feed into the TP4056 Type-C input** so the 1S2P
  pack can be recharged from the vehicle's accessory (cigarette-lighter) socket
- **Part / variant:** HBK Travel Car Charger C102/C103 — **order the `C103 TYPE-C`
  variant only.** The same listing also sells C102 and micro-USB-V8 / iOS builds;
  those connectors will not plug into the TP4056
- **Input:** 12/24V DC from the vehicle accessory socket *(typical for this
  class of travel charger — **UNVERIFIED, not measured on this project**)*
- **Output:** **5V DC on the USB Type-C output** *(confirmed — C103 variant)*,
  dual USB outputs
- **Connection:** **car charger Type-C output → USB-C cable → TP4056 Type-C
  port** (direct 5V charge feed; no wiring into the device harness — external
  accessory only)
- **Current:** charger rating **TBD: verify label/datasheet**; the TP4056 draws
  its ~1A charge current, so any ≥1A USB source is sufficient
- **⚠️ NOT a pack regulator** — it only sources 5V. CC/CV charging, 4.2V
  cut-off and protection stay with the TP4056/DW01
- **⚠️ Charging still requires the main switch ON** (TP4056 B+ sits after SW1)
  — see SETUP.md Step 7 and WIRING.md §6.4
- **⚠️ Vehicle socket:** check whether the socket is ignition-switched or
  always-on; an always-on socket will keep the charger powered when the vehicle
  is parked — unplug when not charging *(NEED USER INPUT)*

#### Boost Converter (XL6009E1 / LM2577-type)
- **Input:** datasheet-dependent — **XL6009E1 is rated 3.6V–36V input** (XLSEMI
  datasheet; some older/duplicate datasheets state 5V–32V). **LM2577: 3.5V–40V**
  (TI). *A Li-Po runs 3.0–4.2V, so operation below ~3.6V is NOT guaranteed*
  — verify your actual module starts and regulates down to 3.5V, otherwise use a
  boost rated for ≥3V input (e.g. MT3608, 2–24V). **NEED USER INPUT / measure.**
- **Output:** Adjustable, target 5.0V for ESP32 VIN
- **Current:** Up to 3-4A (XL6009) / ~2A (LM2577) — **TBD: verify module rating**
- **Adjustment:** Multi-turn potentiometer
- **⚠️ MUST measure output before connecting ESP32**

#### Buzzer
- **Type:** Active (built-in oscillator) or Passive (needs PWM) — **TBD: verify**
- **Voltage:** 3.3V or 5V — **TBD: verify**
- **Current:** ~10-30 mA

#### Miniature Rocker Switch
- **Rating:** 3A+ @ 250VAC / 6A @ 125VAC (typical)
- **DC rating:** Verify for 5V/3.7V switching
- **Mounting:** Panel mount

#### IP65 Enclosure (160×160×90 mm)
- **Material:** ABS
- **Lid:** Transparent
- **Sealing:** Gasket + screws
- **Mounting:** Internal standoffs or external brackets
- **Cable glands:** TBD for antenna/power entry

---

### 2.3 Items Requiring Physical Verification

| Item | Must Verify |
|------|-------------|
| ESP32 exact model & pinout | Pin numbers, USB-UART chip, strapping pins |
| SIM808 UART logic level | VMCU jumper/resistor setting for 3.3V |
| Boost converter max current | Module label / datasheet |
| Buzzer type & voltage | Active vs passive, 3.3V vs 5V |
| SIM808 antenna connectors | SMA / U.FL / onboard |
| 1000µF capacitor polarity | BAT+ = positive, BAT- = negative |
| TP4056 charge current resistor | Actual charge current |
| Car charger variant & output | Received **C103 TYPE-C** (not C102/V8/iOS); **5V on its Type-C output at the TP4056 port** under load |
| Li-Po cell protection | Protected vs bare cells |
| Grove LED pinout | Signal pin position |

---

### 2.4 NOT in BOM (Phase 2 Only)

- Push button
- Microphone / voice module
- LCD / OLED
- Additional MCU (Raspberry Pi, Arduino)
