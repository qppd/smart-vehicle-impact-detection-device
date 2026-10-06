# SETUP.md
## IoT-Based Vehicle Impact Detection System

---

### 1.1 Project Overview

Develop an IoT-based vehicle impact/collision detection device installed inside a vehicle. The device detects suspected impacts, acquires GPS location, and sends an automated SMS emergency notification to a registered EMS/hospital recipient.

### 1.2 Phase 1 — Current Prototype (Implemented)

| Feature | Status |
|---------|--------|
| Impact detection via MPU6050 accelerometer | Implemented |
| Impact processing via ESP32 | Implemented |
| GPS coordinate acquisition via SIM808 | Implemented |
| SMS emergency notification via SIM808 | Implemented |
| Green LED — normal/monitoring status | Implemented |
| Red LED — impact/emergency indication | Implemented |
| Buzzer — audible warning | Implemented |
| 15-second emergency confirmation window | Implemented |
| Automatic SMS if not cancelled | Implemented |
| GPS location in emergency SMS | Implemented |
| Operation from Li-Po battery system | Implemented |
| Weatherproof enclosure installation | Implemented |

### 1.3 Phase 2 — Future Enhancements (Deferred)

| Feature | Status | Notes |
|---------|--------|-------|
| Push-button cancellation | **NOT INCLUDED** | Marked TBD — hardware not supplied |
| Voice recognition/cancellation | **NOT INCLUDED** | Marked TBD — hardware not supplied |
| Microphone/voice command | **NOT INCLUDED** | No microphone in BOM |

Architecture must allow Phase 2 additions without redesigning the entire system.

### 1.4 Currently NOT Included (Do Not Add)

- LCD / OLED display
- Raspberry Pi / Arduino Mega
- ADXL345
- Telegram bot
- Wi-Fi cloud dashboard
- Mobile application / web application
- Database / external server
- Speaker / microphone
- Voice-recognition module
- Push button

These may only appear in a clearly labeled **Future Enhancement / Phase 2** section.

### 1.5 SIM Card

- TNT SIM card, registered
- Use for GSM/SMS testing
- Do NOT request or expose the user's actual phone number
- Actual SMS behavior depends on: SIM registration, network availability, GSM coverage, account load/service availability, SIM compatibility, antenna connection, module power stability

### 1.6 Core System Flow

```
POWER ON
  ↓
SYSTEM INITIALIZATION
  ↓
CHECK MPU6050
  ↓
CHECK SIM808
  ↓
CHECK GPS/GSM STATUS
  ↓
NORMAL MONITORING
  ↓
GREEN LED ON
  ↓
CONTINUOUS MPU6050 MONITORING
  ↓
IMPACT DETECTED?
  ├── NO → Continue monitoring
  │
  └── YES
      ↓
     RED LED ON
      ↓
     BUZZER WARNING
      ↓
     15-SECOND CONFIRMATION WINDOW
      ↓
     CANCELLATION?
     ├── YES → Reset to monitoring
     └── NO
         ↓
       GET GPS LOCATION
         ↓
       FORMAT EMERGENCY SMS
         ↓
       SEND SMS TO REGISTERED EMS/HOSPITAL NUMBER
         ↓
       EMERGENCY STATE
         ↓
       LOG/INDICATE RESULT
```

### 1.7 Phase 1 Cancellation Mechanism

Since push button and voice recognition are deferred, the 15-second confirmation window in Phase 1 has NO user cancellation input. The system MUST send the emergency SMS after the countdown expires.

Mark the human cancellation interface as: **TBD / Phase 2 — Push Button or Voice Command**

Do not invent an alternative cancellation mechanism.

### 1.8 Impact Detection Requirements

Do NOT use a simple `if acceleration > threshold` without justification.

Required considerations:
- X/Y/Z acceleration components
- Acceleration magnitude (sqrt(x² + y² + z²))
- Gravitational acceleration component
- Sudden acceleration spike detection
- Sampling rate configuration
- Filtering (noise reduction)
- Threshold calibration
- Persistence check
- Cooldown period
- False positive prevention
- Vehicle vibration vs. real impact distinction
- Sensor orientation compensation

### 1.9 GPS Requirements

- GPS initialization sequence
- GPS enable command
- GPS fix detection
- Latitude/longitude extraction
- GPS accuracy indication
- Timeout handling
- No-fix handling
- SMS behavior when GPS unavailable

### 1.10 SMS Requirements

- Configurable emergency SMS format
- Registered EMS/hospital recipient (configurable, NOT hard-coded in public docs)
- GPS location in SMS
- Google Maps link
- Handle GPS unavailable case

### 1.11 UART Requirements

- ESP32 ↔ SIM808 via TTL UART (NOT USB)
- Avoid UART pins needed by MPU6050 I²C
- Avoid ESP32 bootstrapping pins
- Avoid USB/programming serial pins
- Final pin assignment in a table

### 1.12 Power Architecture

- 2× 3.7V 2000 mAh Li-Po in parallel (1S2P)
- TP4056 for charging (NOT a 5 V regulator)
- XL6009E1 / LM2577 boost to 5.0 V
- Boost output to ESP32 VIN
- SIM808 powered from battery (3.5–4.2 V)
- 1000µF 25V electrolytic capacitor across SIM808 BAT+/BAT- to smooth 2A TX bursts
- Verify before connecting: polarity, voltage, ground common

### 1.13 Engineering Standards

- CORRECTNESS → SAFETY → BUILDABILITY → TESTABILITY → CLARITY
- TBD for unknown values — VERIFY FROM ACTUAL HARDWARE
- No silent assumptions
- Configurable parameters, not hard-coded magic numbers
- Non-blocking firmware design (millis()-based)
- Fail-safe behavior where practical

---

### 1.3 Hardware Assembly (from 13_HARDWARE_ASSEMBLY.md)

#### Step 1: Prepare Li-Po Battery Pack
1. Verify both batteries are 3.7V nominal and voltage-matched (<0.1V diff).
2. Place batteries side-by-side with terminals aligned.
3. **Parallel connection:** 
   - Connect both positive terminals together with a short wire (red).
   - Connect both negative terminals together with a short wire (black).
4. Insulate connections with heat shrink or electrical tape.
5. Label the pack: BAT+ (red), BAT- (black).

#### Step 2: Install Main Power Switch
1. Cut the BAT+ wire from the battery pack.
2. Solder one end to one terminal of the rocker switch.
3. Solder the other end of the BAT+ wire (from battery) to the other switch terminal.
4. Verify switch opens and closes the circuit with multimeter.

#### Step 3: Connect Boost Converter Input
1. From the switch output (BAT+ after switch), connect to boost converter VIN (red wire).
2. Connect BAT- (black) to boost converter GND (black wire).
3. Use 18-20 AWG wire for these high-current paths.

#### Step 4: Set Boost Converter Output Voltage
1. **Do not connect to ESP32 yet.**
2. Power on the system (close switch).
3. Measure boost converter output with multimeter.
4. Adjust potentiometer until output reads 5.0V (±0.1V).
5. Power off before proceeding.

#### Step 5: Connect ESP32 Power
1. Connect boost converter VOUT to ESP32 VIN (red wire).
2. Connect boost converter GND to ESP32 GND (black wire).
3. Verify 5.0V at ESP32 VIN with multimeter.

#### Step 6: Power SIM808 Directly from Battery
1. Connect BAT+ (before switch) to SIM808 BAT+ (red wire).
2. Connect BAT- (before switch) to SIM808 BAT- (black wire).
   - *Alternative:* Connect after switch if you want SIM808 to power off with main switch.
   - *Recommendation:* Connect before switch so SIM808 can operate during charging (if desired).
3. Use 18-20 AWG wire.
4. **Install 1000µF 25V capacitor** across BAT+ and BAT- at SIM808 terminals:
   - Positive lead to BAT+, negative lead to BAT-
   - This suppresses voltage droop during 2A TX bursts

#### Step 7: Connect TP4056 for Charging
1. Connect battery pack BAT+ to TP4056 BAT+ (red wire).
2. Connect battery pack BAT- to TP4056 BAT- (black wire).
3. Verify TP4056 charging LED behavior when USB-C is plugged in.

#### Step 8: MPU6050 Wiring
1. Connect MPU6050 VCC to ESP32 3.3V (red wire).
2. Connect MPU6050 GND to ESP32 GND (black wire).
3. Connect MPU6050 SDA to ESP32 GPIO21 (yellow wire).
4. Connect MPU6050 SCL to ESP32 GPIO22 (yellow wire).
5. Add 4.7kΩ pull-up resistors from SDA to 3.3V and SCL to 3.3V.

#### Step 9: SIM808 UART Wiring
1. Connect SIM808 TXD to ESP32 GPIO16 (green wire).
2. Connect SIM808 RXD to ESP32 GPIO17 (green wire).
3. Connect SIM808 GND to ESP32 GND (black wire).
4. **Set SIM808 UART logic level to 3.3V:** 
   - Check SIM808 board for VMCU setting (jumper or resistor).
   - Adjust to output 3.3V logic.

#### Step 10: LED and Buzzer Wiring
1. **Green LED:**
   - SIG to ESP32 GPIO18 (green wire)
   - VCC to ESP32 3.3V (red wire)
   - GND to ESP32 GND (black wire)
2. **Red LED:**
   - SIG to ESP32 GPIO19 (red wire)
   - VCC to ESP32 3.3V (red wire)
   - GND to ESP32 GND (black wire)
3. **Buzzer:**
   - SIG to ESP32 GPIO23 (blue wire)
   - VCC to ESP32 3.3V (red wire) [if buzzer needs separate power]
   - GND to ESP32 GND (black wire)
   - *Note:* Some buzzers are powered directly from the signal pin.

#### Step 11: Common Ground Verification
1. With power off, measure resistance between:
   - Battery negative
   - ESP32 GND
   - SIM808 GND
   - MPU6050 GND
   - LED GNDs
   - Buzzer GND
2. All should read <0.1Ω (continuity).

#### Step 12: Neat and Secure Wiring
1. Trim excess wire.
2. Use zip ties or adhesive mounts to secure wires.
3. Route wires away from moving parts and heat sources.
4. Label critical connections if helpful.

---

### 1.4 Enclosure Layout (from 14_ENCLOSURE_LAYOUT.md)

#### 14.1 Enclosure Specifications

- **Model:** IP65 ABS Weatherproof Enclosure
- **Dimensions:** 160 × 160 × 90 mm (external)
- **Internal usable space:** ~150 × 150 × 80 mm (accounting for wall thickness and lid)
- **Lid:** Transparent (allows LED visibility)
- **Material:** ABS plastic
- **Sealing:** Gasket + 4-6 screws
- **Color:** Typically gray or black

#### 14.2 Component Dimensions (Approximate)

| Component | Approx. Size (mm) | Notes |
|-----------|-------------------|-------|
| ESP32 Dev Board | 55 × 28 × 15 | With USB and headers |
| MPU6050 Breakout | 20 × 15 × 5 | Small, can be mounted anywhere |
| SIM808 Module | 50 × 30 × 10 | With pins, needs antenna connectors |
| SIM808 GPS Antenna | 25 × 25 × 8 | Ceramic patch, cable ~100mm |
| SIM808 GSM Antenna | 15 × 15 × 5 | Small PCB, cable ~100mm |
| Li-Po Pack (2× parallel) | 70 × 35 × 10 | Two cells side-by-side |
| TP4056 Board | 30 × 20 × 8 | USB-C on edge |
| Boost Converter | 45 × 25 × 15 | With pot and heatsink |
| Grove LEDs (each) | 20 × 20 × 12 | Panel-mount, 8mm hole |
| Buzzer | 12 × 12 × 8 | Panel-mount, 6-8mm hole |
| Rocker Switch | 15 × 10 × 12 | Panel-mount, 12mm hole |
| Wiring harness | Variable | Allow space for cable routing |

#### 14.3 Recommended Layout (Top-Down View)

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENCLOSURE (160×160)                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    GPS ANTENNA (top center)             │   │
│  │  ⬤                                                       │   │
│  │                                                           │   │
│  │  ┌─────────┐                     ┌─────────┐             │   │
│  │  │  ESP32  │                     │ SIM808  │             │   │
│  │  │  (center)                   │(right)  │             │   │
│  │  └─────────┘                     └────┬────┘             │   │
│  │         │                              │                 │   │
│  │  ┌──────▼──────┐         ┌─────────────▼──────┐          │   │
│  │  │  Boost Conv. │         │  Battery Pack     │          │   │
│  │  │  (left)      │         │  (bottom left)    │          │   │
│  │  └─────────────┘         └────────────────────┘          │   │
│  │                                                           │   │
│  │  ┌─────┐  ┌─────┐     ┌─────┐  ┌─────┐  ┌──────┐         │   │
│  │  │ GLED│  │ RLED│     │ BUZ │  │ USB │  │ SWITCH│        │   │
│  │  │(left)   │(right)    │(mid)  │(mid)  │(right)        │   │
│  │  └─────┘  └─────┘     └─────┘  └─────┘  └──────┘         │   │
│  │                                                           │   │
│  │                    GSM ANTENNA (right side)               │   │
│  │                                                         ⬤  │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### 14.4 Detailed Placement Coordinates

Using bottom-left of internal space as origin (0,0), dimensions in mm:

| Component | X (mm) | Y (mm) | Z/Height | Mounting Method |
|-----------|--------|--------|----------|-----------------|
| ESP32 | 50 | 50 | 15 | Standoffs (3mm) + screws |
| MPU6050 | 60 | 70 | 5 | Double-sided tape on ESP32 or separate standoffs |
| SIM808 | 110 | 50 | 10 | Standoffs (3mm) + screws |
| GPS Antenna | 75 | 80 | 8 | Adhesive on lid (top center) |
| GSM Antenna | 135 | 40 | 5 | Adhesive on right wall |
| Battery Pack | 20 | 20 | 10 | Velcro straps or foam block |
| Boost Converter | 20 | 60 | 15 | Standoffs + screws |
| TP4056 | 20 | 80 | 8 | Hot glue or standoffs |
| Green LED | 40 | 130 | 12 | Panel-mount in lid/wall |
| Red LED | 100 | 130 | 12 | Panel-mount in lid/wall |
| Buzzer | 70 | 130 | 8 | Panel-mount in lid/wall |
| Rocker Switch | 130 | 130 | 12 | Panel-mount in right wall |
| USB-C (ESP32) | 50 | 10 | - | Cutout in left wall |
| USB-C (TP4056) | 20 | 10 | - | Cutout in left wall (or shared) |

#### 14.5 Mounting Hole Pattern

**Internal standoffs (M3, 10mm height):**
- ESP32: 4 holes matching board (typically 2.54mm pitch, ~48×25mm)
- SIM808: 2-4 holes matching module
- Boost Converter: 2-4 holes
- TP4056: 2 holes

**Panel cutouts (in walls/lid):**
- Green LED: 8mm hole (check Grove LED spec)
- Red LED: 8mm hole
- Buzzer: 6-8mm hole + sound holes (3-4mm × 4-6)
- Rocker Switch: 12mm × 8mm rectangular (or per switch spec)
- USB-C ports: 9mm × 7mm rectangular
- Antenna cables: 3-4mm holes for cable pass-through

#### 14.6 Antenna Placement Critical Rules

1. **GPS Antenna (ceramic patch):**
   - Must have clear view of sky
   - Mount on **TOP of enclosure** (lid center)
   - Keep away from metal (>20mm)
   - Cable: route away from power wires
   - Ground plane: ideally on metal plate, but ABS is fine

2. **GSM Antenna:**
   - Mount on **side wall** (vertical orientation best)
   - Keep away from battery and boost converter (>30mm)
   - Cable: keep straight, avoid sharp bends
   - No metal shielding in radiation pattern

3. **General:**
   - Antennas must NOT touch each other
   - Maintain ≥30mm separation between GPS and GSM antennas
   - Keep cables away from high-current power wires

#### 14.7 LED Visibility

- **Transparent lid:** LEDs visible through lid if mounted on PCB facing up
- **Alternative:** Mount LEDs on side wall for external visibility
- **Recommended:** Panel-mount Grove LEDs on **front face** (160mm side)
  - Green left, Red right, Buzzer center
  - Clear labels: "NORMAL", "EMERGENCY"

#### 14.8 Buzzer Audibility

- **Sound holes:** Drill 4-6 holes of 3mm diameter in front of buzzer
- **Cover:** Optional fine mesh to keep insects out while allowing sound
- **Orientation:** Buzzer face toward holes, back against wall
- **Test:** Verify >80dB at 1m in free air

#### 14.9 Switch Access

- **Main power switch:** Mount on **right side wall** (easy access)
- **Orientation:** UP = ON (standard), DOWN = OFF
- **Label:** "POWER" with ON/OFF markings
- **Sealing:** Use switch with built-in boot or add rubber cover

#### 14.10 USB-C Access

Two USB-C ports needed:
1. **ESP32 USB-C:** For programming/debug (left side wall, lower)
2. **TP4056 USB-C:** For charging (left side wall, lower, near ESP32)

**Option A:** Two separate cutouts
**Option B:** Single cutout with internal USB-C hub (not recommended for waterproofing)

**Recommendation:** Two separate cutouts with cable glands or rubber grommets.

#### 14.11 Thermal Management

| Component | Heat Generation | Mitigation |
|-----------|-----------------|------------|
| Boost Converter | Moderate (1-2W) | Mount on wall with air gap, add small heatsink |
| TP4056 | Low-Moderate (1W charging) | Air gap, not enclosed in foam |
| SIM808 | Burst (up to 2W during TX) | Mount with air gap, avoid insulation |
| ESP32 | Low (0.5W) | Normal convection |
| Battery | Low (unless fast charge) | Not in foam, allow convection |

**Ventilation:** Small vent holes (2-3mm) with hydrophobic membrane (Gore-Tex type) if available, or labyrinth path to maintain IP65.

#### 14.12 Cable Routing

```
Power cables (18-20 AWG): Thick, short, along walls
Signal cables (22-28 AWG): Bundled, away from power
Antenna cables: Separate, straight, no loops
USB cables: Strain relief at exit point
```

**Cable management:**
- Use adhesive cable tie mounts on enclosure walls
- Leave service loops (50mm) at each component
- Color-code or label wires at both ends

#### 14.13 Assembly Sequence

1. **Drill all holes** in enclosure (verify layout first with paper template)
2. **Install panel components** (LEDs, buzzer, switch, USB cutouts with glands)
3. **Mount standoffs** for PCBs
4. **Install antennas** (GPS on lid, GSM on side)
5. **Wire power system** (battery → switch → boost → ESP32; battery → SIM808; battery → TP4056)
6. **Wire signals** (I²C, UART, GPIO)
7. **Test outside enclosure** (fully functional)
8. **Install PCBs** on standoffs
9. **Secure battery** with Velcro/foam
10. **Route and tie wires**
11. **Apply sealant** to all penetrations
12. **Close lid**, torque screws evenly
13. **Final test** with enclosure closed

#### 14.14 Waterproofing Checklist (IP65)

- [ ] All panel cutouts sealed (O-rings, gaskets, or silicone)
- [ ] Cable glands tightened on all external cables
- [ ] Lid gasket clean and undamaged
- [ ] Screws torqued evenly (compress gasket uniformly)
- [ ] Antenna cables sealed at exit
- [ ] No gaps at corners
- [ ] Test: spray water (IPX5) - no ingress
- [ ] Test: dust (IP5X) - no ingress

#### 14.15 Serviceability Features

- **Battery removal:** Velcro straps, accessible through lid
- **SIM card access:** SIM808 accessible or SIM slot on enclosure exterior
- **ESP32 USB:** Accessible through side port
- **TP4056 USB:** Accessible for charging
- **Debug header:** Optional 4-pin header inside for serial debug

#### 14.16 Dimension Verification

**Before drilling, verify:**
- [ ] Actual enclosure internal dimensions (may differ from spec)
- [ ] Actual component dimensions (measure with calipers)
- [ ] Standoff heights clear all components
- [ ] Lid closes without pressing on tall components
- [ ] Antennas don't hit lid when closed

**Tolerance:** Allow 2-3mm clearance on all sides.

#### 14.17 Phase 2 Provisions

Reserve space for:
- Push button: 8mm hole on front panel
- Voice module: ~30×20mm PCB space near ESP32
- Additional status LED: extra panel hole

#### 14.18 Bill of Materials for Enclosure

| Item | Qty | Spec |
|------|-----|------|
| IP65 Enclosure | 1 | 160×160×90mm, transparent lid |
| M3 Standoffs (10mm) | 12-16 | Male-female + screws |
| M3 Screws | 20+ | Pan head |
| Cable Glands (M12) | 4-6 | For USB, antenna, power |
| Rubber Grommets | 4 | For LED/buzzer holes |
| Silicone Sealant | 1 tube | Electronics-safe (neutral cure) |
| Velcro Straps | 4 | Battery retention |
| Double-sided Foam Tape | 1 roll | Component mounting |
| Hydrophobic Vent | 1-2 | Optional, for pressure equalization |

#### 14.19 Layout Validation Checklist

- [ ] All components fit with 2mm clearance
- [ ] Antennas have clear radiation paths
- [ ] LEDs visible through lid/wall
- [ ] Buzzer audible through sound holes
- [ ] Switch accessible and operable
- [ ] USB ports accessible
- [ ] Battery removable
- [ ] SIM card accessible (if needed)
- [ ] Wiring has strain relief
- [ ] Heat-generating components have airflow
- [ ] Center of gravity stable
- [ ] Mounting holes for vehicle attachment (external brackets)

#### 14.20 Final Notes

- Dimensions are **approximate** — measure your actual components.
- Create a **paper/cardboard mockup** before drilling.
- Take photos of final layout for future reference.
- Document any deviations from this guide.

---

### 1.5 Chapter 1–3 Prototype Guide (from 17_CHAPTER_1_TO_3_PROTOTYPE_GUIDE.md)

#### 17.1 Purpose

This guide explains what can be demonstrated for the **Chapter 1–3 defense/prototype presentation**. It clearly identifies implemented features vs. future enhancements.

#### 17.2 Implemented Features (Phase 1 — Ready for Demo)

| Feature | Status | Demo Method |
|---------|--------|-------------|
| **Impact Detection** | ✅ Implemented | Shake/tap device; red LED + buzzer activate |
| **MPU6050 Processing** | ✅ Implemented | Serial monitor shows raw/dynamic acceleration |
| **ESP32 Processing** | ✅ Implemented | State machine visible on serial output |
| **Green LED (Normal)** | ✅ Implemented | ON during monitoring |
| **Red LED (Impact)** | ✅ Implemented | ON + flashing during countdown |
| **Buzzer Warning** | ✅ Implemented | Beeps on impact, pattern during countdown |
| **GPS Acquisition** | ✅ Implemented | After 15s, GPS fix shown on serial |
| **SIM808 SMS** | ✅ Implemented | SMS sent to configured number |
| **Emergency Countdown** | ✅ Implemented | 15-second timer visible on serial |
| **GPS in SMS** | ✅ Implemented | Lat/long + Google Maps link in SMS |
| **No-Fix Handling** | ✅ Implemented | "GPS UNAVAILABLE" in SMS if no fix |

#### 17.3 Features NOT Implemented (Phase 2 — Future Enhancement)

| Feature | Status | Notes |
|---------|--------|-------|
| **Push-Button Cancellation** | ❌ **Phase 2** | Hardware not supplied; marked TBD in code |
| **Voice Recognition** | ❌ **Phase 2** | Hardware not supplied; no microphone in BOM |
| **Voice Cancellation** | ❌ **Phase 2** | Depends on voice recognition module |
| **Mobile App / Dashboard** | ❌ **Not Planned** | Not in scope |
| **Telegram / Cloud** | ❌ **Not Planned** | No internet connectivity |
| **LCD / OLED Display** | ❌ **Not Planned** | Not in BOM |

#### 17.4 Demo Script for Defense

**Duration:** ~5-10 minutes

##### 1. Power-On & Initialization (1 min)
- Close main power switch
- Show serial monitor: BOOT → SELF_TEST → MONITORING
- Green LED ON, Red LED OFF, buzzer silent

##### 2. Normal Monitoring (30 sec)
- Device at rest on table
- Serial shows acceleration magnitude ~1.0g
- Dynamic magnitude ~0.0g
- No impact detection

##### 3. Impact Detection (1 min)
- Tap/shake device firmly (simulate impact)
- Red LED turns ON immediately
- Buzzer emits warning beep
- Serial shows: "IMPACT DETECTED!"
- State: IMPACT_DETECTED → CONFIRMATION_WINDOW

##### 4. 15-Second Countdown (1 min)
- Red LED flashes every 500ms
- Buzzer beeps every 500ms
- Serial shows countdown timer: 14s, 13s... 0s
- **Phase 1:** No cancellation possible (no button/voice)

##### 5. GPS Acquisition (30 sec - 1 min)
- After 15s, system enters GPS_ACQUISITION
- Serial shows GPS polling
- Wait for fix (outdoor recommended)
- Show latitude/longitude on serial

##### 6. SMS Emergency Notification (30 sec)
- System enters SMS_SENDING
- Serial shows AT command sequence
- Recipient phone receives SMS
- Show SMS content:
  ```
  EMERGENCY ALERT
  Possible vehicle impact detected.
  
  Location:
  Latitude: 14.XXXXXX
  Longitude: 121.XXXXXX
  
  Google Maps:
  https://maps.google.com/?q=14.XXXXXX,121.XXXXXX
  
  Please check the vehicle/occupant immediately.
  ```

##### 7. Emergency State (30 sec)
- System enters EMERGENCY state
- Periodic emergency beep pattern
- Red LED ON
- Only power cycle resets (Phase 1)

##### 8. Reset Demonstration (30 sec)
- Power cycle (switch OFF then ON)
- System returns to MONITORING
- Green LED ON, Red LED OFF

#### 17.5 Technical Points to Emphasize

1. **Impact Detection Algorithm:**
   - Not simple threshold; uses dynamic magnitude (gravity removed)
   - Persistence check (3 consecutive samples)
   - Cooldown period (3s) prevents re-trigger

2. **State Machine Architecture:**
   - Non-blocking design using `millis()`
   - Clear states: BOOT → MONITORING → IMPACT_DETECTED → CONFIRMATION_WINDOW → GPS_ACQUISITION → SMS_SENDING → EMERGENCY

3. **Power Architecture:**
   - 1S2P Li-Po parallel (4000 mAh total)
   - TP4056 for charging only (NOT 5V regulator)
   - Boost converter (XL6009) to 5.0V for ESP32 VIN
   - SIM808 powered directly from battery (3.7V)

4. **Communication:**
   - MPU6050 via I²C (GPIO21/22)
   - SIM808 via UART2 (GPIO16/17) at 115200 baud
   - SMS text mode, GPS via AT+CGNSINF

5. **Safety Features:**
   - 15-second cancellation window (Phase 1: auto-send)
   - GPS timeout (30s) with fallback message
   - SMS retry (3 attempts)
   - Watchdog timer enabled

#### 17.6 Hardware Walkthrough

Show the physical prototype:

1. **Enclosure** (IP65 ABS, 160×160×90mm)
2. **Main power switch** (right side)
3. **LED indicators** (Green = Normal, Red = Emergency)
4. **Buzzer** (audible alert)
5. **GPS antenna** (top of enclosure)
6. **GSM antenna** (side of enclosure)
7. **USB-C ports** (ESP32 programming + TP4056 charging)
7. **Internal components:**
   - ESP32 38-pin dev board
   - MPU6050 breakout
   - SIM808 module
   - Boost converter
   - TP4056 charger
   - 2× Li-Po 2000mAh (parallel)

#### 17.7 Calibration Demonstration

Show calibration data:
- Spreadsheet with test scenarios (stationary, driving, bumps, impacts)
- Maximum normal dynamic magnitude measured
- Threshold set with safety margin
- Zero false positives during test drive

#### 17.8 Known Limitations (Honest Assessment)

1. **Phase 1 has no user cancellation** — SMS always sends after 15s
2. **GPS requires sky view** — indoor demo will show "GPS UNAVAILABLE"
3. **SIM card must have load/coverage** — SMS fails without network
4. **Threshold calibrated for specific vehicle** — needs re-tuning per vehicle
5. **No battery monitoring in Phase 1** — add in Phase 2
6. **No deep sleep** — continuous operation, ~200-300mA draw

#### 17.9 Q&A Preparation

| Question | Answer |
|----------|--------|
| "Why no cancellation button?" | Deferred to Phase 2; hardware not provided; architecture ready for it |
| "What if GPS fails?" | Sends SMS with "GPS UNAVAILABLE" after 30s timeout |
| "How accurate is impact detection?" | Calibrated from real driving data; ~0 false positives in test |
| "Can it detect rollover?" | Not in Phase 1; gyroscope data available for Phase 2 |
| "Battery life?" | ~12-24h continuous; depends on GPS/SMS frequency |
| "Why SMS not internet?" | Reliable in areas without data coverage; works on basic GSM |
| "Commercial viability?" | Prototype only; needs certification, ruggedization, regulatory |

#### 17.10 Presentation Tips

- Keep serial monitor visible on screen
- Have a second phone ready to receive SMS
- Pre-charge batteries before demo
- Test GPS fix beforehand (cold start can take 30s)
- Bring backup USB cable and power bank
- Prepare slides showing block diagram, state machine, calibration data

#### 17.11 Summary

**Phase 1 delivers a complete, working prototype** that:
- Detects impacts with calibrated algorithm
- Warns with LED + buzzer for 15 seconds
- Acquires GPS location
- Sends emergency SMS with location
- Handles GPS/network failures gracefully

**Phase 2 additions** (push button, voice) are clearly separated and architecturally supported.

---

### 1.6 Final Build Checklist (from 19_FINAL_BUILD_CHECKLIST.md)

#### 19.1 Hardware Checklist

| # | Component | Verified | Notes |
|---|-----------|----------|-------|
| H1 | ESP32 38-pin Dev Board | ☐ | Model: __________ |
| H2 | MPU6050 Breakout (soldered) | ☐ | I²C address 0x68 |
| H3 | SIM808 Module + Antennas | ☐ | GPS + GSM antennas |
| H4 | Grove LED Green | ☐ | Panel mounted |
| H5 | Grove LED Red | ☐ | Panel mounted |
| H6 | Li-Po 3.7V 2000mAh ×2 | ☐ | Matched voltage <0.1V |
| H7 | TP4056 USB-C Charger | ☐ | Protection IC verified |
| H8 | Boost Converter XL6009E1 | ☐ | Set to 5.0V |
| H9 | Buzzer | ☐ | Type: active/passive |
| H10 | Mini Rocker Switch | ☐ | DC rated |
| H11 | Tinned Copper Wire (spool) | ☐ | Various AWG |
| H12 | USB Type-C Cable | ☐ | Data + power |
| H13 | DT-830D Multimeter | ☐ | Calibrated |
| H14 | IP65 Enclosure 160×160×90 | ☐ | Gasket + screws |
| H15 | Dupont Wire Kit | ☐ | Prototyping |
| H16 | TNT SIM Card (registered) | ☐ | Balance + coverage |

#### 19.2 Power Architecture Checklist

| # | Check | Verified | Reading |
|---|-------|----------|---------|
| P1 | Battery pack voltage | ☐ | _____ V |
| P2 | Cell voltage match | ☐ | Δ = _____ V |
| P3 | Boost converter output (no load) | ☐ | _____ V |
| P4 | Boost output under ESP32 load | ☐ | _____ V |
| P5 | SIM808 supply voltage | ☐ | _____ V |
| P6 | TP4056 charge voltage | ☐ | _____ V |
| P7 | TP4056 charge current | ☐ | _____ mA |
| P8 | Common ground continuity | ☐ | _____ Ω |
| P9 | Main switch operation | ☐ | ☐ ON ☐ OFF |
| P10 | No short circuits (power rails) | ☐ | |

#### 19.3 Wiring Checklist

| # | Connection | Verified | Wire AWG/Color |
|---|------------|----------|----------------|
| W1 | Battery BAT+ → Switch → Boost VIN / SIM808 BAT+ / TP4056 BAT+ | ☐ | _____ |
| W2 | Battery BAT- → Boost GND / SIM808 BAT- / TP4056 BAT- | ☐ | _____ |
| W3 | Boost VOUT → ESP32 VIN | ☐ | _____ |
| W4 | Boost GND → ESP32 GND | ☐ | _____ |
| W5 | ESP32 3.3V → MPU6050 VCC | ☐ | _____ |
| W6 | ESP32 GND → MPU6050 GND | ☐ | _____ |
| W7 | ESP32 GPIO21 → MPU6050 SDA | ☐ | _____ |
| W8 | ESP32 GPIO22 → MPU6050 SCL | ☐ | _____ |
| W9 | 4.7kΩ pull-up SDA → 3.3V | ☐ | _____ |
| W10 | 4.7kΩ pull-up SCL → 3.3V | ☐ | _____ |
| W11 | ESP32 GPIO16 → SIM808 TXD | ☐ | _____ |
| W12 | ESP32 GPIO17 → SIM808 RXD | ☐ | _____ |
| W13 | ESP32 GND → SIM808 GND | ☐ | _____ |
| W14 | SIM808 VMCU → 3.3V (logic level) | ☐ | _____ |
| W15 | ESP32 GPIO18 → Green LED SIG | ☐ | _____ |
| W16 | ESP32 3.3V → Green LED VCC | ☐ | _____ |
| W17 | ESP32 GND → Green LED GND | ☐ | _____ |
| W18 | ESP32 GPIO19 → Red LED SIG | ☐ | _____ |
| W19 | ESP32 3.3V → Red LED VCC | ☐ | _____ |
| W20 | ESP32 GND → Red LED GND | ☐ | _____ |
| W21 | ESP32 GPIO23 → Buzzer SIG | ☐ | _____ |
| W22 | ESP32 3.3V → Buzzer VCC (if needed) | ☐ | _____ |
| W23 | ESP32 GND → Buzzer GND | ☐ | _____ |

#### 19.4 Software Checklist

| # | Item | Verified | Notes |
|---|------|----------|-------|
| S1 | ESP32 board package installed | ☐ | Version: _____ |
| S2 | Firmware compiled (no errors) | ☐ | |
| S3 | EMS_PHONE_NUMBER configured | ☐ | +63__________ |
| S4 | GPIO pins match hardware | ☐ | Verified in config.h |
| S5 | MPU6050 init successful | ☐ | Serial: "MPU6050 OK" |
| S6 | SIM808 AT commands respond | ☐ | Serial: "SIM808 OK" |
| S7 | SIM808 CPIN READY | ☐ | |
| S8 | SIM808 CREG registered | ☐ | |
| S9 | GPS acquisition works | ☐ | Fix: Y/N, Time: _____ s |
| S10 | Test SMS sent successfully | ☐ | Received: Y/N |
| S11 | Impact threshold calibrated | ☐ | Value: _____ g |
| S12 | 15s countdown works | ☐ | Verified on serial |
| S13 | False positive test passed | ☐ | 0-1 per 10 km |
| S14 | GPS unavailable handled | ☐ | SMS: "GPS UNAVAILABLE" |
| S15 | SMS retry mechanism tested | ☐ | 3 retries |
| S16 | Emergency state entered | ☐ | After SMS |
| S17 | Power cycle resets to monitoring | ☐ | |

#### 19.5 Mechanical Checklist

| # | Item | Verified | Notes |
|---|------|----------|-------|
| M1 | Enclosure layout verified | ☐ | Components fit |
| M2 | Antenna placement optimal | ☐ | GPS top, GSM side |
| M3 | GPS antenna sky view clear | ☐ | No metal obstruction |
| M4 | GSM antenna clear | ☐ | Away from battery |
| M5 | LED visibility confirmed | ☐ | Through lid/wall |
| M6 | Buzzer audibility >80dB@1m | ☐ | Sound holes drilled |
| M7 | Main switch accessible | ☐ | Labeled ON/OFF |
| M8 | USB-C ports accessible | ☐ | ESP32 + TP4056 |
| M9 | Cable glands installed | ☐ | IP65 maintained |
| M10 | Cable strain relief | ☐ | Zip ties/mounts |
| M11 | Battery secured (Velcro) | ☐ | No movement |
| M12 | Components on standoffs | ☐ | No shorts |
| M13 | Silicone sealant applied | ☐ | All penetrations |
| M14 | Enclosure closes fully | ☐ | No pressure on components |
| M15 | Screws torqued evenly | ☐ | Gasket compressed |

#### 19.6 Integration Test Results

| Test | Pass/Fail | Notes |
|------|-----------|-------|
| T1: Normal monitoring (green LED) | ☐ ☐ | |
| T2: Impact detection (tap test) | ☐ ☐ | |
| T3: 15s countdown (LED + buzzer) | ☐ ☐ | |
| T4: GPS fix acquired | ☐ ☐ | Time: _____ s |
| T5: Emergency SMS with GPS | ☐ ☐ | Format verified |
| T6: GPS unavailable SMS | ☐ ☐ | Indoor test |
| T7: Hard braking no trigger | ☐ ☐ | 3+ tests |
| T8: Speed bump no trigger | ☐ ☐ | 3+ tests |
| T9: Pothole no trigger | ☐ ☐ | 3+ tests |
| T10: Power cycle reset | ☐ ☐ | |
| T11: Battery life estimate | ☐ ☐ | _____ hours |
| T12: Enclosure IP65 (spray test) | ☐ ☐ | Optional |

#### 19.7 Documentation Checklist

| # | Document | Created | Reviewed |
|---|----------|---------|----------|
| D1 | SETUP.md | ☐ | ☐ |
| D2 | BOM.md | ☐ | ☐ |
| D3 | SYSTEM-ARCHITECTURE.md | ☐ | ☐ |
| D4 | BLOCK-DIAGRAM.md | ☐ | ☐ |
| D5 | FLOWCHART.md | ☐ | ☐ |
| D6 | WIRING.md | ☐ | ☐ |
| D7 | STACKS.md | ☐ | ☐ |
| D8 | FIRMWARE.md | ☐ | ☐ |
| D9 | TESTING.md | ☐ | ☐ |
| D10 | TROUBLESHOOTING.md | ☐ | ☐ |

#### 19.8 Calibration Data Record

| Scenario | Max Dynamic (g) | RMS (g) | 99th %ile (g) | Date |
|----------|-----------------|---------|---------------|------|
| Stationary | | | | |
| Engine Idle | | | | |
| Smooth Road | | | | |
| Rough Road | | | | |
| Speed Bump | | | | |
| Hard Braking | | | | |
| Hard Accel | | | | |
| Sharp Turn | | | | |
| **Max Normal** | | | | |
| **Controlled Impact (avg)** | | | | |

**Final IMPACT_THRESHOLD_G = _______ g** (max_normal + 0.5g margin)

#### 19.9 Known Issues / Deviations

| # | Issue | Mitigation / Note |
|---|-------|-------------------|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |

#### 19.10 Sign-Off

**Built by:** _________________________ **Date:** ___________

**Tested by:** _________________________ **Date:** ___________

**Approved for Defense:** ☐ Yes ☐ No

**Comments:**
________________________________________________________________
________________________________________________________________
________________________________________________________________

#### 19.11 Quick Reference: Critical Values

| Parameter | Value | Location |
|-----------|-------|----------|
| ESP32 GPIO (SDA/SCL) | 21 / 22 | config.h |
| ESP32 GPIO (UART2 RX/TX) | 16 / 17 | config.h |
| ESP32 GPIO (Green/Red/Buzzer) | 18 / 19 / 23 | config.h |
| IMPACT_THRESHOLD_G | _______ g | config.h (calibrated) |
| IMPACT_PERSISTENCE | 3 samples | config.h |
| IMPACT_COOLDOWN_MS | 3000 ms | config.h |
| CONFIRMATION_WINDOW_MS | 15000 ms | config.h |
| GPS_TIMEOUT_MS | 30000 ms | config.h |
| EMS_PHONE_NUMBER | +63__________ | config.h |
| Boost Converter Output | 5.00 V | Measured |
| Battery Pack (1S2P) | 3.7V / 4000mAh | BOM |
| SIM808 Supply | 3.5-4.2 V | Direct from battery |

#### 19.12 First Test to Perform After Assembly

**Electrical Tests (E1-E11)** (Section 15.2) before any firmware upload.

**Then:** Upload calibration sketch (Section 12.3) → Record stationary data → Verify MPU6050 reads ~1g on one axis.

**Then:** Upload full firmware → Open serial monitor → Power on → Verify state sequence: BOOT → SELF_TEST → MONITORING → Green LED ON.

---

*End of SETUP.md*
