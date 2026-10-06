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
| 18 | Resettable Fuse 3A | 1 | 3A hold, auto-recover | Battery positive line protection | TBD |

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
- **GSM:** Quad-band 850/900/1800/1900 MHz
- **GPS:** L1 C/A, 42 channels, -160 dBm tracking, -143 dBm cold start
- **TTFF:** ~30s cold, ~1s hot
- **Accuracy:** <2.5m CEP
- **UART:** TTL, RXD/TXD/VMCU
- **VMCU:** Sets UART logic level (1.25V / 3.3V / 5V) — **SET TO 3.3V FOR ESP32**
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

#### Boost Converter (XL6009E1 / LM2577-type)
- **Input:** 3.0–32V (depends on exact module)
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
|| 1000µF capacitor polarity | BAT+ = positive, BAT- = negative |
| Resettable fuse | Rating, hold current, trip time |
| TP4056 charge current resistor | Actual charge current |
| Li-Po cell protection | Protected vs bare cells |
| Grove LED pinout | Signal pin position |

---

### 2.4 NOT in BOM (Phase 2 Only)

- Push button
- Microphone / voice module
- LCD / OLED
- Additional MCU (Raspberry Pi, Arduino)
