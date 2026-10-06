# Block Diagrams
## IoT-Based Vehicle Impact Detection System

---

### 4.1 System Block Diagram (Top-Level)

```mermaid
graph TB
    subgraph SENSOR["SENSOR"]
        MPU6050["MPU6050<br/>Accelerometer + Gyroscope"]
    end

    subgraph MCU["MCU"]
        ESP32["ESP32 Development Board<br/>Main MCU"]
    end

    subgraph COMM["COMMUNICATION"]
        SIM808["SIM808 Module<br/>GSM + GPS"]
    end

    subgraph PERIPH["PERIPHERALS"]
        GREEN["🟢 Green LED"]
        RED["🔴 Red LED"]
        BUZZ["🔊 Buzzer"]
    end

    subgraph POWER["POWER SUBSYSTEM"]
        BAT["Li-Po Pack<br/>1S2P 3.7V 4000mAh"]
        BOOST["Boost Converter<br/>3.7V → 5V"]
        TP4056["TP4056 Charger<br/>USB-C"]
    end

    MPU6050 -- "I²C" --> ESP32
    ESP32 -- "UART2" --> SIM808
    SIM808 -- "GSM SMS" --> GREEN
    SIM808 -- "GSM SMS" --> RED
    BUZZ --> ESP32
    BAT --> BOOST
    BAT --> TP4056
    BAT --> SIM808
    BOOST --> ESP32
    TP4056 -. "Charging only" .-> BAT
```

---

### 4.2 Power Block Diagram

```mermaid
graph TB
    BAT["Li-Po Pack<br/>3.7V 4000mAh"]
    SW["Main Switch"]
    BOOST["Boost Converter<br/>3.7V → 5V"]
    TP4056["TP4056<br/>USB-C Charger"]
    ESP32["ESP32 VIN<br/>5V"]
    SIM808["SIM808<br/>BAT+"]

    BAT --> SW
    SW --> BOOST
    SW --> SIM808
    SW --> TP4056
    BOOST --> ESP32
    TP4056 -. "Charging" .-> BAT
```

---

### 4.3 Signal Flow Diagram

```mermaid
flowchart LR
    subgraph MONITOR["MONITORING"]
        MPU["MPU6050"]
    end

    subgraph DETECT["IMPACT DETECTION"]
        IMPACT["Impact Detector<br/>Threshold + Persistence"]
    end

    subgraph CONFIRM["CONFIRMATION WINDOW"]
        COUNTDOWN["15s Countdown"]
    end

    subgraph GPS["GPS ACQUISITION"]
        GPS["AT+CGNSINF Poll"]
    end

    subgraph SMS["SMS SENDING"]
        SEND["AT+CMGS"]
    end

    subgraph EMERGENCY["EMERGENCY STATE"]
        ALERT["RED LED + BUZZER"]
    end

    MPU --> IMPACT
    IMPACT --> COUNTDOWN
    COUNTDOWN --> GPS
    GPS --> SEND
    SEND --> ALERT
```

---

### 4.4 Communication Architecture Diagram

```mermaid
graph TB
    ESP32["ESP32 GPIO"]
    I2C["I²C"]
    UART["UART2"]

    I2C -- "SDA/SCL<br/>GPIO21/22" --> MPU6050["MPU6050"]
    UART -- "TXD/RXD<br/>GPIO16/17" --> SIM808["SIM808"]
    ESP32 -- "GPIO18" --> GREEN["Green LED"]
    ESP32 -- "GPIO19" --> RED["Red LED"]
    ESP32 -- "GPIO23" --> BUZZ["Buzzer"]

    subgraph NOTES["NOTES"]
        VMCU["VMCU = 3.3V"]
        BAUD["UART2 @ 115200 bps"]
    end
```

---

### 4.5 Enclosure Block Diagram

```mermaid
graph TB
    subgraph TOP["TOP - GPS ANTENNA"]
        GPS_ANT["GPS Antenna<br/>Ceramic Patch"]
    end

    subgraph FRONT["FRONT PANEL"]
        GLED["Green LED"]
        RLED["Red LED"]
        BZZ["Buzzer"]
        SW["Power Switch"]
        USB1["USB-C ESP32"]
    end

    subgraph INSIDE["INSIDE"]
        ESP32["ESP32"]
        SIM808["SIM808"]
        MPU6050["MPU6050"]
        BOOST["Boost Converter"]
        TP4056["TP4056"]
        BAT["Battery Pack"]
    end

    subgraph BACK["BACK PANEL"]
        GSM_ANT["GSM Antenna"]
        USB2["USB-C TP4056"]
        GL["Cable Glands"]
    end

    GPS_ANT --> SIM808
    GSM_ANT --> SIM808
    BAT --> SIM808
    BAT --> BOOST
    BAT --> TP4056
    BOOST --> ESP32
    ESP32 --> MPU6050
    ESP32 --> SIM808
```

---

### 4.6 Phase 2 Extension Block Diagram

```mermaid
graph TB
    PHASE1["Phase 1: Core"]
    BUTTON["Push Button<br/>GPIO4"]
    VOICE["Voice Module<br/>I²C 0x30"]
    ADC["Battery Monitor<br/>Voltage Divider"]

    PHASE1 --> BUTTON
    PHASE1 --> VOICE
    PHASE1 --> ADC

    subgraph CONFIRM["CONFIRMATION WINDOW"]
        CHECK_CANCEL{"Cancel?"}
    end

    BUTTON --> CHECK_CANCEL
    VOICE --> CHECK_CANCEL
    CHECK_CANCEL -->|YES| RESET["RESET → MONITORING"]
    CHECK_CANCEL -->|NO| GPS["GPS ACQUISITION"]
```

---

## Summary

- Clear state machine with 9 states
- Modular architecture (sensor, communication, peripheral modules)
- Non-blocking using millis()
- Configurable parameters in config.h
- Fault tolerance for sensor/communication failures
- Phase 2 extension points documented
