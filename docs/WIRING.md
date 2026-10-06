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
    ESP32["ESP32 38-pin"]
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
| **MPU6050** | SDA | GPIO21 | I²C data (open-drain, 4.7k pull-up to 3.3V) |
|  | SCL | GPIO22 | I²C clock (open-drain, 4.7k pull-up to 3.3V) |
| **SIM808** | TXD (module TX → ESP32 RX) | GPIO16 | UART2_RX |
|  | RXD (module RX ← ESP32 TX) | GPIO17 | UART2_TX |
| **Green LED** | Signal | GPIO18 | Output — ON = normal monitoring |
| **Red LED** | Signal | GPIO19 | Output — ON = impact detected / emergency |
| **Buzzer** | Signal | GPIO25 | Output — active buzzer (HIGH = tone) |
| **Main Switch** | Power | N/A | Hardware switch on main positive line |

**[WARNING] VERIFY:** These GPIO numbers are for a generic ESP32 38-pin board. You MUST verify the exact pinout of your ESP32 board before finalizing connections.

**[WARNING] WROVER WARNING:** If your ESP32 module is WROVER (has PSRAM), GPIO16 and GPIO17 are internally bonded to the PSRAM chip and NOT available externally. Use alternate UART2 pins (e.g., GPIO25/26) or switch to a WROOM module.

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

### 6.4 Power Wiring

```mermaid
graph TB
    BAT["Li-Po Pack<br/>BAT+ red, BAT- black"]
    SW["Main Switch"]
    SIM808["SIM808 BAT+"]
    TP4056["TP4056 BAT+"]
    BOOST["Boost Converter VIN"]

    BAT --> SW
    SW --> SIM808
    SW --> TP4056
    SW --> BOOST
    BOOST --> ESP32["ESP32 VIN 5.0V"]
```

**Main Switch:** In the positive line from battery pack (before boost converter and SIM808).

---

### 6.5 Complete Connection Table

```mermaid
graph LR
    subgraph POWER["POWER"]
        P1["BAT+ → SIM808 BAT+"]
        P2["BAT- → SIM808 BAT-"]
        P3["BAT+ → TP4056 BAT+"]
        P4["BAT- → TP4056 BAT-"]
        P5["BAT+ → Boost VIN"]
        P6["BAT- → Boost GND"]
        P7["Boost VOUT → ESP32 VIN"]
        P8["Boost GND → ESP32 GND"]
        P9["Switch → Battery BAT+"]
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
|| P1 | Li-Po Pack BAT+ | SIM808 BAT+ | 18 AWG red | 3.5–4.2V | Power SIM808 | Add 1000µF cap at SIM808 end |
|| P2 | Li-Po Pack BAT- | SIM808 BAT- | 18 AWG black | 0V | Ground for SIM808 | |
|| P3 | Li-Po Pack BAT+ | TP4056 BAT+ | 18 AWG red | 3.5–4.2V | Charging input | |
|| P4 | Li-Po Pack BAT- | TP4056 BAT- | 18 AWG black | 0V | Ground for TP4056 | |
|| P5 | Li-Po Pack BAT+ | Boost VIN | 18 AWG red | 3.5–4.2V | Boost input | |
|| P6 | Li-Po Pack BAT- | Boost GND | 18 AWG black | 0V | Boost ground | |
|| P7 | Boost VOUT | ESP32 VIN | 20 AWG red | 5.0V set | ESP32 power | **Measure before connecting** |
|| P8 | Boost GND | ESP32 GND | 20 AWG black | 0V | ESP32 ground | |
|| P9 | Main Switch | Battery BAT+ | 18 AWG red | 3.7V | Switch positive line | |
|| P10 | 1000µF 25V Cap + | SIM808 BAT+ | Short red wire | 3.5–4.2V | Capacitor positive | Low ESR preferred |
|| P11 | 1000µF 25V Cap - | SIM808 BAT- | Short black wire | 0V | Capacitor negative | Across power input |

#### ESP32–MPU6050 I²C

| # | FROM | TO | WIRE | VOLTAGE | PURPOSE | NOTES |
|---|------|----|------|---------|---------|-------|
| I2C1 | ESP32 GPIO21 | MPU6050 SDA | 22 AWG yellow | 3.3V | I²C data | Add 4.7kΩ pull-up |
| I2C2 | ESP32 GPIO22 | MPU6050 SCL | 22 AWG yellow | 3.3V | I²C clock | Add 4.7kΩ pull-up |
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
|| BZ1 | ESP32 GPIO25 | Buzzer SIG | 22 AWG blue | 3.3V | Buzzer control | **Verify buzzer type** |
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
    I -->|YES| J["MPU6050 pull-ups 4.7kΩ?"]
    J -->|NO| ADDP["Add pull-up resistors"]
    J -->|YES| K["Antennas connected?"]
    K -->|NO| CONN["Connect antennas"]
    K -->|YES| READY["[OK] Ready to power on"]
```

- [ ] Battery voltage: 3.5–4.2V
- [ ] Boost output: 5.0V ±0.1V (no load)
- [ ] ESP32 VIN: 5.0V
- [ ] SIM808 voltage: = battery voltage
- [ ] Ground continuity <0.1Ω everywhere
- [ ] No short circuits on power rails
- [ ] Correct polarity on all polarized components
- [ ] SIM808 VMCU set to 3.3V
- [ ] MPU6050 pull-ups installed (4.7kΩ)
- [ ] Antennas connected

---

### 6.9 Enclosure Wiring Notes

- Antenna routing: Keep GPS/GSM antennas away from noisy power wires
- Strain relief: Cable ties or clamps at entry points
- Serviceability: Leave slack for battery removal
- Heat: Keep boost converter and TP4056 away from enclosed batteries
- See `SETUP.md` Section 13 for enclosure layout
