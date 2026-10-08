# System Architecture
## IoT-Based Vehicle Impact Detection System

---

### 3.1 High-Level Overview

The system is an embedded IoT device that:
1. Monitors vehicle motion via MPU6050 accelerometer
2. Detects significant acceleration spikes (potential impact)
3. Enters a 15-second confirmation window (visual/audible alerts)
4. If not cancelled, acquires GPS from SIM808
5. Formats and sends emergency SMS via SIM808
6. Stays in EMERGENCY state (red LED + periodic beeps); in Phase 1 only a
   power cycle returns the device to monitoring

All processing occurs on the ESP32. No external server/cloud required.

---

### 3.2 System Block Diagram

```mermaid
graph TB
    subgraph SENSOR["SENSOR"]
        MPU6050["MPU6050 Accelerometer"]
    end

    subgraph MCU["MCU"]
        ESP32["ESP32 Development Board"]
    end

    subgraph COMM["COMMUNICATION"]
        SIM808["SIM808 GSM/GPS Module"]
    end

    subgraph PERIPH["PERIPHERALS"]
        GREEN["Green LED"]
        RED["Red LED"]
        BUZZ["Buzzer"]
    end

    subgraph POWER["POWER"]
        BAT["Li-Po Pack 1S2P"]
        BOOST["Boost Converter 3.7V→5V"]
        TP4056["TP4056 Charger"]
    end

    MPU6050 -- "I²C" --> ESP32
    ESP32 -- "UART2" --> SIM808
    SIM808 -- "GSM SMS" --> PHONE["Recipient phone"]
    ESP32 --> GREEN
    ESP32 --> RED
    ESP32 --> BUZZ
    BAT --> BOOST
    BAT --> TP4056
    BAT --> SIM808
    BOOST --> ESP32
```

---

### 3.3 Data Flow

```mermaid
flowchart LR
    A["Power ON"] -->    B["System Init"]
    B --> C["MPU6050 Init"]
    C --> D["SIM808 Init"]
    D --> E["GPS Power On (AT+CGNSPWR=1)"]
    E --> F["Main Loop"]
    F --> G["Read MPU6050"]
    G --> H["Compute Magnitude"]
    H --> I["Filter/Validate"]
    I --> J{"Impact Detected?"}
    J -- "NO" --> F
    J -- "YES" --> K["Red LED ON"]
    K --> L["Buzzer Warning"]
    L --> M["15s Countdown"]
    M --> N{"Cancelled?"}
    N -- "YES" --> F
    N -- "NO" --> O["GPS Acquisition"]
    O --> P{"GPS Fix?"}
    P -- "NO" --> Q["SMS: GPS Unavailable"]
    P -- "YES" --> R["Format SMS"]
    Q --> R
    R --> S["Send SMS"]
    S --> T["Emergency State"]
    T --> U["Reset?"]
    U -- "YES" --> F
    U -- "NO" --> T
```

---

### 3.4 State Machine

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

### 3.5 Communication Interfaces

```mermaid
graph TB
    ESP32_GPIO["ESP32 GPIO"]
    I2C["I²C Bus"]
    UART["UART2"]

    I2C -- "SDA/SCL" --> MPU6050["MPU6050"]
    UART -- "TXD/RXD" --> SIM808["SIM808"]
    ESP32_GPIO -- "GPIO18" --> GREEN["Green LED"]
    ESP32_GPIO -- "GPIO19" --> RED["Red LED"]
    ESP32_GPIO -- "GPIO25" --> BUZZ["Buzzer"]
```

---

### 3.6 Timing Characteristics

```mermaid
timeline
    title System Timing Overview
    section Normal Monitoring
      MPU6050 Read : 10ms interval : 100 Hz sampling
    section Impact Detected
      Countdown : 15 seconds : 500ms flash interval
      GPS Acquisition : 30 seconds max : poll every 2s
      SMS Sending : 10s timeout : retry 3 times
    section Emergency
      Beep Pattern : 5s interval : long-short-long
```

---

### 3.7 Fault Tolerance

```mermaid
graph LR
    A["Sensor/Module"] --> B{"Response?"}
    B -- "OK" --> C["Continue"]
    B -- "FAIL" --> D["ERROR State"]
    D --> E["Fast Blink LED"]
    E --> F["Reset?"]
    F -- "YES" --> A
    F -- "NO" --> G["Await User"]
```

---

### 3.8 Power Management

```mermaid
graph TB    BAT["Li-Po Pack 1S2P"] --> SW["Main Switch"]
    SW --> TP4056["TP4056 B+"]
    SW --> SIM808["SIM808 BAT+"]
    TP4056 --> BOOST["Boost Converter<br/>(from OUT±)"]
    BOOST --> ESP32["ESP32 VIN 5V"]
    TP4056 -. "USB-C Charging" .-> SW
    BAT --> CAP["1000µF 25V Capacitor<br/>(at SIM808)"]
    CAP --> SIM808
```

---

### 3.9 Phase 2 Extension Points

```mermaid
graph TB
    P1["Phase 1: Core"]
    P2A["Phase 2A: Push Button"]
    P2B["Phase 2B: Voice Recognition"]
    P2C["Phase 2C: Battery Monitor"]
    P2D["Phase 2D: SMS Config"]

    P1 --> P2A
    P1 --> P2B
    P1 --> P2C
    P1 --> P2D
```

---

## Summary

- Clear state machine with 9 states
- Modular architecture (sensor, communication, peripheral modules)
- millis()-based state timing (blocking waits remain inside AT-command and beep
  helpers — see FIRMWARE.md §8.3)
- Configurable parameters in config.h
- Boot-time fault detection (MPU6050/self-test failure → ERROR); runtime SMS
  failures are logged and the alert sequence still completes
- Phase 2 extension points documented
