# Testing & Validation
## IoT-Based Vehicle Impact Detection System

---

### 9.1 Testing Philosophy

1. **Electrical tests** (before assembly)
2. **Module tests** (each subsystem individually)
3. **Integrated tests** (full system function)
4. **Calibration** (tune thresholds using real data)
5. **Environmental tests** (if needed)

---

### 9.2 Electrical Tests (Before Assembly)

```mermaid
flowchart TD
    START["🔍 START"]
    START --> E1["E1: Visual inspection"]
    E1 --> E2["E2: Battery voltage 3.5-4.2V"]
    E2 --> E3["E3: Cell match <0.1V"]
    E3 --> E4["E4: Boost input = battery"]
    E4 --> E5["E5: Boost output 5.0V"]
    E5 --> E6["E6: ESP32 VIN 5.0V"]
    E6 --> E7["E7: SIM808 = battery"]
    E7 --> E8["E8: TP4056 output"]
    E8 --> E9["E9: Ground <0.1Ω"]
    E9 --> E10["E10: Polarity correct"]
    E10 --> E11["E11: Switch function"]
    E11 --> ALLPASS["✅ ALL PASS"]
    E11 --> FAIL["❌ FAIL → Fix"]
    FAIL --> E1
```

| Test # | Description | Expected Result | Pass/Fail |
|--------|-------------|-----------------|-----------|
| E1 | Visual inspection of all components | No damage, no shorts | ☐ |
| E2 | Battery voltage (each cell) | 3.5–4.2V | ☐ |
| E3 | Battery voltage match | <0.1V difference | ☐ |
| E4 | Boost converter input (no load) | = battery voltage | ☐ |
| E5 | Boost converter output (adjusted) | 5.0V ±0.1V | ☐ |
| E6 | ESP32 VIN (with ESP32 connected) | 5.0V (under load) | ☐ |
| E7 | SIM808 voltage at BAT | = battery voltage | ☐ |
| E8 | TP4056 output (during charging) | ~4.2V, ~1A | ☐ |
| E9 | Ground continuity (all GND points) | <0.1Ω | ☐ |
| E10 | Polarity check | Correct (+/–) on all connections | ☐ |
| E11 | Switch function | Open = off, Closed = on | ☐ |

---

### 9.3 Module Tests

#### ESP32 Module Test
```mermaid
flowchart LR
    UPLOAD["Upload blink sketch"] --> GREEN["Green LED blinks?"]
    GREEN -->|NO| CHECK_GREEN["Check GPIO18 wiring"]
    GREEN -->|YES| RED["Red LED blinks?"]
    RED -->|NO| CHECK_RED["Check GPIO19 wiring"]
    RED -->|YES| BUZZ["Buzzer beeps?"]
    BUZZ -->|NO| CHECK_BUZZ["Check GPIO23 wiring"]
    BUZZ -->|YES| SERIAL["Serial monitor prints?"]
    SERIAL -->|NO| CHECK_SERIAL["Check USB-UART"]
    SERIAL -->|YES| ESP32_OK["✅ ESP32 OK"]
```

- Upload blink sketch to all configured GPIOs
- Green/red LEDs blink → OK
- Buzzer beeps → OK
- Serial monitor prints → OK

#### MPU6050 Module Test
```mermaid
flowchart LR
    SCAN["I²C Scanner"] --> WHO["WHO_AM_I = 0x68?"]
    WHO -->|NO| CHECK_I2C["Check SDA/SCL/pull-ups"]
    WHO -->|YES| DATA["Read accel values"]
    DATA --> MOVE["Move sensor"]
    MOVE --> CHANGES["Values change?"]
    CHANGES -->|NO| CHECK_MPU["Check wiring/power"]
    CHANGES -->|YES| REST["At rest ~1g on Z?"]
    REST -->|NO| CALIB["Calibrate bias"]
    REST -->|YES| MPU6050_OK["✅ MPU6050 OK"]
```

- Upload I²C scanner + accelerometer read sketch
- WHO_AM_I returns 0x68
- Serial monitor shows x/y/z acceleration values
- Values change with sensor movement
- At rest, one axis reads ~1g (gravity)

#### SIM808 GSM Module Test
```mermaid
flowchart LR
    AT["AT → OK"] --> CPIN["AT+CPIN? → READY"]
    CPIN --> CREG["AT+CREG? → registered"]
    CREG --> CSQ["AT+CSQ → signal strength"]
    CSQ --> SIM808_OK["✅ SIM808 OK"]
```

- Send AT commands: `AT` → `OK`, `AT+CPIN?` → `READY`, `AT+CREG?` → registered
- `AT+CSQ` → signal strength

#### SIM808 SMS Test
```mermaid
flowchart LR
    CMGF["AT+CMGF=1"] --> CMGS["AT+CMGS='+your_number'"]
    CMGS --> MSG["Type 'Test' + Ctrl+Z"]
    MSG --> CHECK["Check phone for SMS"]
    CHECK -->|NO| FIX["Check network/SIM"]
    CHECK -->|YES| SMS_OK["✅ SMS OK"]
```

- `AT+CMGF=1` → OK
- `AT+CMGS="+your_number"` → `>`
- Type "Test" + Ctrl+Z (0x1A)
- Check phone for "Test" SMS

#### SIM808 GPS Module Test
```mermaid
flowchart LR
    PWR["AT+CGNSPWR=1"] --> WAIT["Wait 30s cold start"]
    WAIT --> INF["AT+CGNSINF"]
    INF --> FIX{"Fix status = 1?"}
    FIX -->|NO| ANT["Check antenna/sky view"]
    FIX -->|YES| GPS_OK["✅ GPS OK"]
```

- `AT+CGNSPWR=1` → OK
- Wait 30s (cold start)
- `AT+CGNSINF` → check fix status, latitude, longitude

---

### 9.4 Integrated Tests

```mermaid
flowchart TD
    POWER_ON["Power on"] --> MONITOR["🟢 Green LED ON"]
    MONITOR --> TAP["Tap/shake device"]
    TAP --> RED_ON["🔴 Red LED ON"]
    RED_ON --> COUNT["15s countdown"]
    COUNT --> GPS["📡 GPS acquisition"]
    GPS --> SMS["📤 SMS sent"]
    SMS --> RECIPIENT["📱 Recipient receives SMS"]
    RECIPIENT --> FORMAT{"Format correct?"}
    FORMAT -->|NO| FIX_SMS["Fix SMS template"]
    FORMAT -->|YES| T1["✅ T1: Normal monitoring"]
    T1 --> T2["T2: Impact detection"]
    T2 --> T3["T3: 15s countdown"]
    T3 --> T4["T4: GPS acquisition"]
    T4 --> T5["T5: Emergency SMS"]
    T5 --> T6["T6: GPS unavailable"]
    T6 --> T7["T7: Hard braking"]
    T7 --> T8["T8: Speed bumps"]
    T8 --> T9["T9: Potholes"]
    T9 --> T10["T10: Power cycle reset"]
    T10 --> T11["T11: Repeated impacts"]
    T11 --> T12["T12: False positive driving"]
```

| Test # | Description | Steps | Expected | Pass/Fail |
|--------|-------------|-------|----------|-----------|
| T1 | Normal monitoring | Power on, observe LEDs | Green ON, Red OFF | ☐ |
| T2 | Impact detection | Tap/shake device | Red ON, buzzer | ☐ |
| T3 | 15s countdown | Watch serial timer | 15s countdown | ☐ |
| T4 | GPS acquisition | Go outside, clear sky | Fix acquired | ☐ |
| T5 | Emergency SMS | Check recipient phone | SMS received | ☐ |
| T6 | GPS unavailable | Indoors, trigger impact | SMS: "GPS UNAVAILABLE" | ☐ |
| T7 | Hard braking | Drive hard brake at 50km/h | No trigger | ☐ |
| T8 | Speed bumps | Drive over bumps | No trigger | ☐ |
| T9 | Potholes | Drive over potholes | No trigger | ☐ |
| T10 | Power cycle reset | Off/on after emergency | Back to monitoring | ☐ |
| T11 | Repeated impacts | Trigger twice quickly | Only one SMS | ☐ |
| T12 | False positive driving | 10km normal driving | 0-1 triggers | ☐ |

---

### 9.5 Calibration Procedure

See full calibration guide in original documentation set:

1. **Mount sensor** in final orientation
2. **Record stationary readings** (engine off, level ground) → compute bias
3. **Record normal driving** (city, highway) → note max dynamic magnitude
4. **Record controlled impacts** (low-speed bumps) → note magnitude
5. **Set IMPACT_THRESHOLD** between normal max and impact min with margin
6. **Validate** with real-world testing on various road surfaces

**Calibration data table:**

| Scenario | Max Dynamic (g) | RMS (g) | 99th %ile (g) |
|----------|-----------------|---------|---------------|
| Stationary | | | |
| Engine Idle | | | |
| Smooth Road | | | |
| Rough Road | | | |
| Speed Bump | | | |
| Hard Braking | | | |
| Hard Accel | | | |
| Sharp Turn | | | |
| **Max Normal** | | | |
| **Controlled Impact (avg)** | | | |

**Recommended threshold:** max_normal + 0.5g margin

---

### 9.6 False Positive Reduction Strategies

```mermaid
flowchart LR
    A["Impact Detection"] --> B{"Threshold tuning"}
    B -->|Too low| C["False positives"]
    B -->|Too high| D["Missed impacts"]
    A --> E{"Persistence check"}
    E -->|N too low| F["Noise triggers"]
    E -->|N too high| G["Delayed detection"]
    A --> H{"Cooldown period"}
    H -->|Too short| I["Re-trigger"]
    H -->|Too long| J["Miss second impact"]
```

| Strategy | Description | Trade-off |
|----------|-------------|-----------|
| Threshold tuning | Set high enough to ignore bumps | Too high → missed impacts |
| Persistence check | Require N consecutive samples | N too high → delayed detection |
| Cooldown period | Ignore re-trigger for 3s | Misses second impact quickly |
| Directional check | Require dominant axis | Less effective for oblique impacts |
| Frequency analysis | Filter low-frequency road noise | More processing required |

---

### 9.7 Test Results Log

| Test # | Description | Result | Notes |
|--------|-------------|--------|-------|
| E1 | Visual inspection | ☐ | |
| E2 | Battery voltage | ☐ | |
| ... | ... | ... | |
| T1 | Normal monitoring | ☐ | |
| T2 | Impact detection | ☐ | |
| ... | ... | ... | |

**Document all test results for defense preparation.**

---

### 9.8 Phase 2 Testing

- Push button cancellation during confirmation window
- Voice recognition cancellation
- Combined operations

---

### 9.9 Summary

1. Perform electrical tests first.
2. Module tests individually.
3. Integrated tests in sequence.
4. Calibrate threshold using real vehicle data.
5. Validate with extensive driving tests.
6. Document all results.
