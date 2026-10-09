# Wiring Guide & Pin Assignments
## IoT-Based Vehicle Impact Detection System

---

### 6.1 Circuit Diagrams

<!-- Wiring Image -->
![Wiring Diagram](../wiring/circuit_image.png)

**Circuit file:** [smart-vehicle-impact-detection-device.ckt](../wiring/smart-vehicle-impact-detection-device.ckt)

**Interactive wiring diagram:** [View on CirKit Designer](https://app.cirkitdesigner.com/project/2e962a5d-9e84-46b6-87e7-2afaff00305b)

---

### 6.2 Pin Assignment

```mermaid
graph LR
    ESP32["ESP32 38-pin Development Board with Terminal Block"]
    MPU6050["MPU6050"]
    SIM808["SIM808"]
    GREEN["Green LED"]
    RED["Red LED"]
    BUZZ["Buzzer"]

    ESP32 -- "GPIO21 SDA" --> MPU6050
    ESP32 -- "GPIO22 SCL" --> MPU6050
    ESP32 -- "GPIO16 RX2" --> SIM808_TX["SIM808 TXD"]
    ESP32 -- "GPIO17 TX2" --> SIM808_RX["SIM808 RXD"]
    ESP32 -- "GPIO18" --> GREEN
    ESP32 -- "GPIO19" --> RED
    ESP32 -- "GPIO25" --> BUZZ

    subgraph NOTES["NOTES"]
        VMCU["VMCU = 3.3V"]
        BAUD["UART2 @ 115200"]
        PULLUP["4.7kΩ pull-ups"]
    end
```

| Peripheral | Signal | ESP32 GPIO | Notes |
|------------|--------|------------|-------|
| **MPU6050** | SDA | GPIO21 | I²C data (open-drain, 4.7kΩ pull-up to 3.3V **if the breakout has none**) |
|  | SCL | GPIO22 | I²C clock (open-drain, 4.7kΩ pull-up to 3.3V **if the breakout has none**) |
| **SIM808** | TXD (module TX → ESP32 RX) | GPIO16 | UART2_RX |
|  | RXD (module RX ← ESP32 TX) | GPIO17 | UART2_TX |
| **Green LED** | Signal | GPIO18 | Output — ON = normal monitoring |
| **Red LED** | Signal | GPIO19 | Output — ON = impact detected / emergency |
| **Buzzer** | Signal | GPIO25 | Output — active buzzer (HIGH = tone) |
| **Main Switch** | Power | N/A | Hardware switch on main positive line |

**[WARNING] VERIFY:** These GPIO numbers are for a generic ESP32 38-pin Development Board with Terminal Block. You MUST verify the exact pinout of your ESP32 board before finalizing connections.

**[WARNING] WROVER WARNING:** If your ESP32 module is WROVER (has PSRAM), GPIO16 and GPIO17 are internally bonded to the PSRAM chip and NOT available externally. Use alternate UART2 pins (e.g., GPIO25/26) or switch to a WROOM module.

**[WARNING] I²C PULL-UPS — CHECK BEFORE ADDING:** Most MPU6050 breakouts (e.g.
GY-521 style boards, typically 2.2–4.7 kΩ) already have pull-ups on SDA/SCL.
**Inspect/measure your board first:**
- Pull-ups already fitted → add nothing (the circuit file adds none).
- No pull-ups → fit external 4.7 kΩ from SDA to 3.3V and SCL to 3.3V.
- Never end up below ~1.0 kΩ total on either line.

---

### 6.3 UART Selection Rationale

```mermaid
graph TB
    UART0["UART0 GPIO1, GPIO3"]
    UART1["UART1 GPIO9, GPIO10"]
    UART2["UART2 GPIO16, GPIO17 [RECOMMENDED]"]

    UART0 -->|"USB-Serial<br/>Programming"| AVOID["[AVOID]"]
    UART1 -->|"Flash chip<br/>ESP32-WROOM"| AVOID
    UART2 -->|"Free on most<br/>dev boards"| USE["[USE]"]
```

- **UART0 (GPIO1, GPIO3):** Reserved for USB-to-UART programming — avoid.
- **UART1 (GPIO9, GPIO10):** Often used for flash chip on ESP32-WROOM modules — avoid.
- **UART2 (GPIO16, GPIO17):** Free on most dev boards, not used for bootstrapping — **recommended**.
- **Baud rate:** 115200 bps (configurable via AT+IPR).

---

### 6.4 Power Wiring (as-built reference — matches the CirKit circuit file)

```mermaid
graph TB
    BAT["Li-Po Pack 1S2P<br/>BAT+ / BAT-"]
    SW["Main Switch SW1"]
    TP4056["TP4056 + DW01<br/>B+ / B− → OUT+ / OUT−"]
    SIM808["SIM808 BAT+ / GND"]
    CAP["1000µF 25V<br/>(at SIM808)"]
    BOOST["Boost Converter<br/>VIN+ / VIN− → VOUT+ / VOUT−"]
    ESP32["ESP32 VIN 5.0V / GND"]

    BAT --> SW
    SW --> TP4056
    SW --> SIM808
    SW --> CAP
    BAT --> TP4056
    BAT --> SIM808
    TP4056 --> BOOST
    BOOST --> ESP32
```

**Topology (verified against `smart-vehicle-impact-detection-device.ckt`):**

1. Pack **BAT+ → SW1**, then the switched rail feeds
   **TP4056 B+** and **SIM808 BAT+** (1000 µF across SIM808 BAT+/BAT−).
2. Pack **BAT−** feeds **TP4056 B−** and **SIM808 GND** (common ground).
3. **TP4056 OUT+ / OUT− → Boost VIN+ / VIN−** (protected charger output feeds the boost).
4. **Boost VOUT+ → ESP32 5V**, **Boost VOUT− → ESP32 GND**.
5. **ESP32 GND → SIM808 GND** explicit wire (UART signal ground — see note below).

**As-built consequences (documented, not changed):**

- **Charging requires SW1 = ON** (TP4056 B+ sits after the switch). *Optional
  improvement:* move the TP4056 B+ lead to the battery side (before SW1) so the pack
  can charge with the switch off.
- The DW01/FS8205 **discharge protection only covers currents returning through
  OUT−** (the boost/ESP32 branch). The SIM808 returns straight to B−, so it is not
  covered by that protection — keep the pack charged and do not leave the device
  draining a flat pack unattended.

**[WARNING] COMMON GROUND:** the CirKit diagram relies on the boost module's
internal VIN−/OUT− connection to join ESP32 ground to battery ground. **Do not rely
on it** — the written table (connection P10 / UART3) requires a direct
**ESP32 GND ↔ SIM808 GND** wire. Without it the UART has no solid reference and
AT communication becomes unreliable.

---

### 6.5 Complete Connection Table

```mermaid
graph LR
    subgraph POWER["POWER"]
        P1["BAT+ → Switch SW1"]
        P2["SW1 → TP4056 B+"]
        P3["SW1 → SIM808 BAT+"]
        P4["BAT- → TP4056 B-"]
        P5["BAT- → SIM808 GND"]
        P6["TP4056 OUT+ → Boost VIN+"]
        P7["TP4056 OUT- → Boost VIN-"]
        P8["Boost VOUT+ → ESP32 VIN"]
        P9["Boost VOUT- → ESP32 GND"]
        P10["ESP32 GND → SIM808 GND"]
        P11["1000µF across SIM808 BAT+/BAT-"]
    end

    subgraph I2C["I²C"]
        I1["GPIO21 → MPU6050 SDA"]
        I2["GPIO22 → MPU6050 SCL"]
        I3["3.3V → MPU6050 VCC"]
        I4["GND → MPU6050 GND"]
    end

    subgraph UART["UART"]
        U1["GPIO16 → SIM808 TXD"]
        U2["GPIO17 → SIM808 RXD"]
        U3["GND → SIM808 GND"]
        U4["3.3V → SIM808 VMCU"]
    end

    subgraph LED["LEDs"]
        L1["GPIO18 → Green LED SIG"]
        L2["3.3V → Green LED VCC"]
        L3["GND → Green LED GND"]
        L4["GPIO19 → Red LED SIG"]
        L5["3.3V → Red LED VCC"]
        L6["GND → Red LED GND"]
    end

    subgraph BUZZ["BUZZER"]
        B1["GPIO25 → Buzzer SIG"]
        B2["3.3V → Buzzer VCC"]
        B3["GND → Buzzer GND"]
    end
```

#### Power Connections

| # | FROM | TO | WIRE | VOLTAGE | PURPOSE | NOTES |
|---|------|----|------|---------|---------|-------|
| P1 | Li-Po Pack BAT+ | Main Switch SW1 | 18 AWG red | 3.5–4.2V | Main power switch | |
| P2 | Switch SW1 output | TP4056 B+ | 18 AWG red | 3.5–4.2V | Charger input | Charge only works with SW1 ON (as-built) |
| P3 | Switch SW1 output | SIM808 BAT+ | 18 AWG red | 3.5–4.2V | Power SIM808 | Add 1000µF cap at SIM808 end |
| P4 | Li-Po Pack BAT- | TP4056 B- | 18 AWG black | 0V | Charger/protect ground | |
| P5 | Li-Po Pack BAT- | SIM808 BAT- | 18 AWG black | 0V | Ground for SIM808 | |
| P6 | TP4056 OUT+ | Boost VIN+ | 18 AWG red | = pack voltage | Protected boost input | **Use OUT, not B+, per circuit file** |
| P7 | TP4056 OUT- | Boost VIN- | 18 AWG black | 0V | Boost ground input | |
| P8 | Boost VOUT+ | ESP32 VIN (5V) | 20 AWG red | 5.0V set | ESP32 power | **Measure before connecting** |
| P9 | Boost VOUT- | ESP32 GND | 20 AWG black | 0V | ESP32 ground | |
| P10 | ESP32 GND | SIM808 GND | 22 AWG black | 0V | **UART signal ground** | Required — omitted from circuit diagram |
| P11 | 1000µF 25V Cap + | SIM808 BAT+ | Short red wire | 3.5–4.2V | TX-burst smoothing | Low ESR preferred |
| P13 | 1000µF 25V Cap - | SIM808 BAT- | Short black wire | 0V | Capacitor negative | Across power input |

#### ESP32–MPU6050 I²C

| # | FROM | TO | WIRE | VOLTAGE | PURPOSE | NOTES |
|---|------|----|------|---------|---------|-------|
| I2C1 | ESP32 GPIO21 | MPU6050 SDA | 22 AWG yellow | 3.3V | I²C data | 4.7kΩ pull-up **if breakout has none** |
| I2C2 | ESP32 GPIO22 | MPU6050 SCL | 22 AWG yellow | 3.3V | I²C clock | 4.7kΩ pull-up **if breakout has none** |
| I2C3 | ESP32 3.3V | MPU6050 VCC | 22 AWG red | 3.3V | Power MPU6050 | |
| I2C4 | ESP32 GND | MPU6050 GND | 22 AWG black | 0V | Ground | |

#### ESP32–SIM808 UART

| # | FROM | TO | WIRE | VOLTAGE | PURPOSE | NOTES |
|---|------|----|------|---------|---------|-------|
| UART1 | ESP32 GPIO16 (RX2) | SIM808 TXD | 22 AWG green | 3.3V | SIM808 → ESP32 | |
| UART2 | ESP32 GPIO17 (TX2) | SIM808 RXD | 22 AWG green | 3.3V | ESP32 → SIM808 | |
| UART3 | ESP32 GND | SIM808 GND | 22 AWG black | 0V | UART ground | |
| UART4 | ESP32 3.3V | SIM808 VMCU | 22 AWG white | 3.3V | UART logic level | **SET TO 3.3V** |

#### ESP32–LEDs

| # | FROM | TO | WIRE | VOLTAGE | PURPOSE | NOTES |
|---|------|----|------|---------|---------|-------|
| LED1 | ESP32 GPIO18 | Green LED SIG | 22 AWG green | 3.3V | Green LED control | |
| LED2 | ESP32 3.3V | Green LED VCC | 22 AWG red | 3.3V | Green LED power | |
| LED3 | ESP32 GND | Green LED GND | 22 AWG black | 0V | Green LED ground | |
| LED4 | ESP32 GPIO19 | Red LED SIG | 22 AWG red | 3.3V | Red LED control | |
| LED5 | ESP32 3.3V | Red LED VCC | 22 AWG red | 3.3V | Red LED power | |
| LED6 | ESP32 GND | Red LED GND | 22 AWG black | 0V | Red LED ground | |

#### ESP32–Buzzer

| # | FROM | TO | WIRE | VOLTAGE | PURPOSE | NOTES |
|---|------|----|------|---------|---------|-------|
| BZ1 | ESP32 GPIO25 | Buzzer SIG | 22 AWG blue | 3.3V | Buzzer control | **Active buzzer module (DC drive) required** |
| BZ2 | ESP32 3.3V | Buzzer VCC | 22 AWG red | 3.3V | Buzzer power | |
| BZ3 | ESP32 GND | Buzzer GND | 22 AWG black | 0V | Buzzer ground | |

---

### 6.6 Wire Gauge Guide

```mermaid
graph LR
    subgraph HIGH["HIGH CURRENT ≥1A"]
        H1["Battery → SIM808<br/>2A peak"]
        H2["Battery → Boost input<br/>2A peak"]
        H1_G["18–20 AWG"]
        H2_G["18–20 AWG"]
    end

    subgraph MED["MEDIUM CURRENT 500mA–1A"]
        M1["Boost → ESP32 VIN<br/>1A"]
        M1_G["20–22 AWG"]
    end

    subgraph LOW["LOW CURRENT <100mA"]
        L1["I²C, UART, GPIO signals"]
        L1_G["22–28 AWG"]
    end

    H1 --> H1_G
    H2 --> H2_G
    M1 --> M1_G
    L1 --> L1_G
```

| Path | Expected Current | Recommended AWG |
|------|-----------------|-----------------|
| Battery → SIM808 | Up to 2A peak | 18–20 AWG |
| Battery → Boost input | Up to 2A peak | 18–20 AWG |
| Boost → ESP32 VIN | Up to 1A | 20–22 AWG |
| I²C, UART, GPIO signals | <100 mA | 22–28 AWG |

---

### 6.7 Voltage Compatibility Checks

```mermaid
graph TB
    MPU["MPU6050 VCC 3.3V"] --> OK1["Compatible"]
    SIM["SIM808 UART 3.3V<br/>VMCU=3.3V"] --> OK2["Compatible"]
    LED["Grove LEDs 3.3V"] --> OK3["Compatible"]
    BUZZ["Buzzer 3.3V/5V"] --> CHECK["Verify rating"]
    ESP32_V["ESP32 VIN 5.0V"] --> OK4["Compatible"]
```

| Device | Voltage | ESP32/GPIO Compatible? |
|--------|---------|------------------------|
| MPU6050 VCC | 3.3V | [OK] Yes |
| SIM808 UART | 3.3V (set VMCU) | [OK] Yes |
| Grove LEDs | 3.3V | [OK] Typically |
| Buzzer | 3.3V/5V | [WARNING] Verify |
| ESP32 VIN | 5.0V | [OK] Set boost to 5.0V |

**If any device is 5V-only:** Use a logic level shifter.

---

### 6.8 Pre-Power-On Checks

```mermaid
flowchart TD
    A["Pre-Power-On Checks"] --> B["Battery voltage 3.5–4.2V?"]
    B -->|NO| FIXB["Charge/replace battery"]
    B -->|YES| C["Boost output 5.0V?"]
    C -->|NO| ADJ["Adjust potentiometer"]
    C -->|YES| D["ESP32 VIN 5.0V?"]
    D -->|NO| CHECK["Check boost output"]
    D -->|YES| E["SIM808 voltage = battery?"]
    E -->|NO| CHECK2["Check wiring"]
    E -->|YES| F["Ground continuity <0.1Ω?"]
    F -->|NO| FIXG["Check ground connections"]
    F -->|YES| G["No short circuits?"]
    G -->|NO| FIXS["Find and fix short"]
    G -->|YES| H["Correct polarity?"]
    H -->|NO| FIXP["Reverse polarity"]
    H -->|YES| I["SIM808 VMCU=3.3V?"]
    I -->|NO| SET["Set VMCU jumper"]
    I -->|YES| J["I²C pull-ups OK?<br/>(onboard or 4.7kΩ added)"]
    J -->|NO| ADDP["Add 4.7kΩ pull-ups"]
    J -->|YES| K["Antennas connected?"]
    K -->|NO| CONN["Connect antennas"]
    K -->|YES| READY["[OK] Ready to power on"]
```

- [ ] Battery voltage: 3.5–4.2V (cells matched <0.1V)
- [ ] Boost output: 5.0V ±0.1V (no load)
- [ ] ESP32 VIN: 5.0V
- [ ] SIM808 voltage: = battery voltage (3.4–4.2V, SIM808 spec is 3.4–4.4V)
- [ ] Ground continuity <0.1Ω everywhere
- [ ] **ESP32 GND ↔ SIM808 GND wire present** (UART signal ground)
- [ ] No short circuits on power rails
- [ ] Correct polarity on all polarized components (incl. 1000µF)
- [ ] SIM808 VMCU set to 3.3V (ESP32 inputs are NOT 5V tolerant)
- [ ] I²C pull-ups: verified present on breakout, or 4.7kΩ added
- [ ] Antennas connected (GSM + GPS)
- [ ] Note: charging works only with the main switch ON (as-built)

---

### 6.9 Enclosure Wiring Notes

- Antenna routing: Keep GPS/GSM antennas away from noisy power wires
- Strain relief: Cable ties or clamps at entry points
- Serviceability: Leave slack for battery removal
- Heat: Keep boost converter and TP4056 away from enclosed batteries
- See `SETUP.md` Section 13 for enclosure layout
