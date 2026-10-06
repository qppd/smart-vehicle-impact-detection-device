# System Flowchart
## IoT-Based Vehicle Impact Detection System

---

### 5.1 Core System Flowchart (Phase 1)

```mermaid
flowchart TD
    POWER["⏚ POWER ON"] --> INIT["🔧 SYSTEM INIT"]
    INIT --> CHECK_MPU["📋 CHECK MPU6050"]
    CHECK_MPU -->|FAIL| ERROR["⚠️ ERROR"]
    CHECK_MPU -->|OK| CHECK_SIM["📋 CHECK SIM808"]
    CHECK_SIM -->|FAIL| ERROR
    CHECK_SIM -->|OK| CHECK_GPS["📋 CHECK GPS/GSM"]
    CHECK_GPS -->|FAIL| ERROR
    CHECK_GPS -->|OK| MONITOR["🟢 NORMAL MONITORING<br/>GREEN LED ON"]

    MONITOR --> READ_MPU["📖 Read MPU6050 @100Hz"]
    READ_MPU --> COMPUTE["📐 Compute Magnitude<br/>√(x²+y²+z²)"]
    COMPUTE --> FILTER["🔇 Filter/Validate"]
    FILTER --> IMPACT{"⚡ Impact Detected?"}
    IMPACT -->|NO| MONITOR

    IMPACT -->|YES| RED_ON["🔴 RED LED ON"]
    RED_ON --> BUZZER_WARN["🔊 BUZZER WARNING"]
    BUZZER_WARN --> COUNTDOWN["⏱️ 15s CONFIRMATION WINDOW"]
    COUNTDOWN --> CANCELLED{"❌ Cancelled?"}

    CANCELLED -->|Phase 2: YES| RESET["🔄 RESET"]
    RESET --> MONITOR

    CANCELLED -->|Phase 1: NO| GPS_START["📡 GET GPS LOCATION"]
    GPS_START --> GPS_POLL{"📍 GPS FIX?"}
    GPS_POLL -->|NO| GPS_UNAVAIL["📱 SMS: GPS UNAVAILABLE"]
    GPS_POLL -->|YES| FORMAT_SMS["✉️ FORMAT EMERGENCY SMS"]
    GPS_UNAVAIL --> FORMAT_SMS
    FORMAT_SMS --> SEND_SMS["📤 SEND SMS"]
    SEND_SMS --> EMERGENCY["🚨 EMERGENCY STATE<br/>RED LED + BUZZER PATTERN"]
    EMERGENCY --> RESET
```

---

### 5.2 State Machine Diagram

```mermaid
stateDiagram-v2
    [*] --> BOOT
    BOOT --> SELF_TEST : init OK
    BOOT --> ERROR : init FAIL

    SELF_TEST --> MONITORING : sensors OK
    SELF_TEST --> ERROR : sensor FAIL

    MONITORING --> IMPACT_DETECTED : impact detected
    IMPACT_DETECTED --> CONFIRMATION_WINDOW : start countdown

    CONFIRMATION_WINDOW --> GPS_ACQUISITION : timeout 15s
    CONFIRMATION_WINDOW --> MONITORING : cancelled Phase 2

    GPS_ACQUISITION --> SMS_SENDING : fix acquired or timeout
    SMS_SENDING --> EMERGENCY : SMS sent
    EMERGENCY --> MONITORING : reset power cycle
    ERROR --> BOOT : reset
```

---

### 5.3 Impact Detection Sub-Flow

```mermaid
flowchart TD
    READ["📖 READ MPU6050<br/>(x, y, z @ 100Hz)"] --> MAG["📐 COMPUTE MAGNITUDE<br/>mag = √(x²+y²+z²)"]
    MAG --> GRAVITY["🔇 REMOVE GRAVITY<br/>dynamic = mag - 1g"]
    GRAVITY --> THRESH{"⚡ DYNAMIC > THRESH?"}
    THRESH -->|NO| MONITOR["Continue monitoring"]
    THRESH -->|YES| PERSIST["📊 PERSISTENCE COUNT++"]
    PERSIST --> COUNT{"COUNT >= 3?"}
    COUNT -->|NO| MONITOR
    COUNT -->|YES| DETECT["⚡ IMPACT DETECTED<br/>(last_impact = millis())"]
    DETECT --> COOLDOWN["⏱️ COOLDOWN 3s"]
    COOLDOWN --> MONITOR
```

---

### 5.4 GPS Acquisition Sub-Flow

```mermaid
flowchart TD
    GPS_ON["📡 GPS POWER ON<br/>AT+CGNSPWR=1"] --> WAIT["⏳ WAIT 2s"]
    WAIT --> POLL["📋 POLL GPS<br/>AT+CGNSINF"]
    POLL --> FIX{"📍 FIX AVAILABLE?<br/>(field 2 = 1)"}
    FIX -->|NO| RETRY{"🔄 RETRY < 10?"}
    RETRY -->|YES| POLL
    RETRY -->|NO| NO_FIX["📱 SMS: GPS UNAVAILABLE"]
    FIX -->|YES| EXTRACT["📝 EXTRACT LAT/LON"]
    EXTRACT --> GPS_OK["✅ GPS FIX OK"]
```

---

### 5.5 SMS Sending Sub-Flow

```mermaid
flowchart TD
    CMGF["📱 SET TEXT MODE<br/>AT+CMGF=1"] --> CMGS["📝 SET RECIPIENT<br/>AT+CMGS="+63...""]
    CMGS --> WAIT_PROMPT{"⏳ WAIT > PROMPT?"}
    WAIT_PROMPT -->|NO| RETRY_SEND["🔄 Retry"]
    WAIT_PROMPT -->|YES| SEND_MSG["✉️ SEND MESSAGE<br/>+ Ctrl+Z 0x1A"]
    SEND_MSG --> WAIT_OK{"⏳ WAIT +CMGS:"}
    WAIT_OK -->|SUCCESS| EMERGENCY["🚨 EMERGENCY STATE"]
    WAIT_OK -->|FAIL| RETRY_COUNT{"🔄 RETRY < 3?"}
    RETRY_COUNT -->|YES| CMGS
    RETRY_COUNT -->|NO| EMERGENCY
```

---

### 5.6 Phase 2 Flowchart Additions

```mermaid
flowchart TD
    CONFIRM["⏱️ CONFIRMATION WINDOW"] --> BUTTON{"❓ PUSH BUTTON?"}
    BUTTON -->|YES| CANCELLED["🔄 CANCELLED → RESET"]
    BUTTON -->|NO| VOICE{"❓ VOICE CANCEL?"}
    VOICE -->|YES| CANCELLED
    VOICE -->|NO| TIMEOUT{"⏱️ TIMEOUT 15s?"}
    TIMEOUT -->|YES| GPS_ACQ["📡 GPS ACQUISITION"]
    TIMEOUT -->|NO| CONFIRM
```

---

### 5.7 Timing Diagram

```mermaid
timeline
    title Impact Detection Timeline
    section Impact Detected
      RED LED ON : 0s
      BUZZER WARNING : 0s - 15s
      COUNTDOWN : 15s
    section GPS Acquisition
      GPS POWER ON : 15s
      POLL GPS : 15s - 45s max
    section SMS Sending
      SEND SMS : 45s+
      EMERGENCY STATE : ongoing
```

---

## Summary

- Clear state machine with 9 states
- Modular architecture (sensor, communication, peripheral modules)
- Non-blocking using millis()
- Configurable parameters in config.h
- Fault tolerance for sensor/communication failures
- Phase 2 extension points documented
