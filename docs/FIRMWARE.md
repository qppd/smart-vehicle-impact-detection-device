# Firmware Implementation
## IoT-Based Vehicle Impact Detection System

---

### 8.1 Complete ESP32 Firmware (Arduino)

Copy the following into your Arduino IDE project. All code is in a single file for simplicity — split into modules as needed.

```cpp
/*
  IoT-Based Vehicle Impact Detection System
  Phase 1 — Impact Detection → 15s Countdown → GPS → SMS
  
  Hardware: ESP32 38-pin, MPU6050, SIM808, Grove LEDs, Buzzer
  
  Author: Senior Embedded Systems Engineer
  Version: 1.0
*/

// ========================
// CONFIGURATION (config.h)
// ========================

// GPIO Assignments — VERIFY WITH YOUR BOARD!
#define PIN_MPU6050_SDA       21
#define PIN_MPU6050_SCL       22
#define PIN_SIM808_RXD        16   // ESP32 UART2 RX (connects to SIM808 TXD)
#define PIN_SIM808_TXD        17   // ESP32 UART2 TX (connects to SIM808 RXD)
#define PIN_GREEN_LED         18
#define PIN_RED_LED           19
#define PIN_BUZZER            25

// MPU6050 Settings
#define MPU6050_ADDR          0x68
#define MPU6050_ACCEL_FS      3    // 0=±2g, 1=±4g, 2=±8g, 3=±16g
#define MPU6050_ACCEL_LSB_G   2048.0  // for ±16g
#define MPU6050_SAMPLE_MS     10   // 100 Hz sampling

// Impact Detection
#define IMPACT_THRESHOLD_G    2.5      // Dynamic acceleration threshold (g) — tune via calibration
#define IMPACT_PERSISTENCE    3        // Consecutive samples over threshold
#define IMPACT_COOLDOWN_MS    3000     // ms to ignore after detection

// System Timing
#define CONFIRMATION_WINDOW_MS  15000  // 15 seconds
#define GPS_TIMEOUT_MS          30000  // 30 seconds for GPS fix
#define SMS_TIMEOUT_MS          10000  // 10 seconds for SMS response
#define SMS_RETRY_COUNT         3
#define UNDERVOLTAGE_MV         3000   // Minimum battery voltage (3.0V)

// Emergency SMS
#define EMS_PHONE_NUMBER      "+63XXXXXXXXXX"  // <-- USER MUST CONFIGURE!
#define SMS_MAX_LENGTH        160

// LED/Buzzer Patterns
#define BUZZER_ACTIVE_HIGH    true   // true = HIGH = on
#define LED_ACTIVE_HIGH       true

// ========================
// MAIN FIRMWARE (main.cpp)
// ========================

#include <Arduino.h>
#include <Wire.h>
#include <HardwareSerial.h>
#include <math.h>

// ---------- State Machine ----------
enum SystemState {
  STATE_BOOT,
  STATE_SELF_TEST,
  STATE_MONITORING,
  STATE_IMPACT_DETECTED,
  STATE_CONFIRMATION_WINDOW,
  STATE_GPS_ACQUISITION,
  STATE_SMS_SENDING,
  STATE_EMERGENCY,
  STATE_ERROR
};

SystemState currentState = STATE_BOOT;
unsigned long stateStartTime = 0;

// ---------- MPU6050 Data ----------
struct AccelData {
  float x, y, z;
  float magnitude;
  float dynamicMagnitude;
};

AccelData accelData;
float accelBias[3] = {0, 0, 0};
bool biasCalibrated = false;
int impactConsecutiveCount = 0;
unsigned long lastImpactTime = 0;

// ---------- SIM808 Communication ----------
HardwareSerial sim808Serial(2);  // UART2
#define SIM808_BAUD 115200

String sim808Buffer;
unsigned long sim808LastCmdTime = 0;
enum Sim808Op {
  SIM808_IDLE,
  SIM808_INIT,
  SIM808_GPS_ON,
  SIM808_GPS_WAIT,
  SIM808_GPS_READ,
  SIM808_SMS_SEND,
  SIM808_SMS_WAIT
};
Sim808Op sim808Op = SIM808_IDLE;
int gpsRetryCount = 0;
int smsRetryCount = 0;
float gpsLatitude = 0, gpsLongitude = 0;
bool gpsFixAvailable = false;

// ---------- Timing ----------
unsigned long lastSampleTime = 0;
unsigned long lastConfirmationBeep = 0;
bool confirmationBeepState = false;

// ========================
// SETUP
// ========================
void setup() {
  Serial.begin(115200);
  delay(100);
  Serial.println("\n=== Vehicle Impact Detection System ===");
  Serial.println("Phase 1 Firmware v1.0");
  
  // GPIO
  pinMode(PIN_GREEN_LED, OUTPUT);
  pinMode(PIN_RED_LED, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);
  digitalWrite(PIN_GREEN_LED, LED_ACTIVE_HIGH ? HIGH : LOW);
  digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? LOW : HIGH);
  digitalWrite(PIN_BUZZER, BUZZER_ACTIVE_HIGH ? LOW : HIGH);
  
  // I2C for MPU6050
  Wire.begin(PIN_MPU6050_SDA, PIN_MPU6050_SCL);
  Wire.setClock(400000);
  
  // UART for SIM808
  sim808Serial.begin(SIM808_BAUD, SERIAL_8N1, PIN_SIM808_RXD, PIN_SIM808_TXD);
  
  // Start state machine
  setState(STATE_BOOT);
}

// ========================
// MAIN LOOP
// ========================
void loop() {
  // State machine handler
  switch (currentState) {
    case STATE_BOOT:            handleBoot();            break;
    case STATE_SELF_TEST:       handleSelfTest();        break;
    case STATE_MONITORING:      handleMonitoring();      break;
    case STATE_IMPACT_DETECTED: handleImpactDetected();  break;
    case STATE_CONFIRMATION_WINDOW: handleConfirmationWindow(); break;
    case STATE_GPS_ACQUISITION: handleGpsAcquisition();  break;
    case STATE_SMS_SENDING:     handleSmsSending();      break;
    case STATE_EMERGENCY:       handleEmergency();        break;
    case STATE_ERROR:           handleError();            break;
  }
  
  // Non-blocking SIM808 processing
  processSIM808();
  
  delay(1);  // yield
}

// ========================
// STATE MACHINE
// ========================
void setState(SystemState newState) {
  // Exit actions
  switch (currentState) {
    case STATE_MONITORING:
      digitalWrite(PIN_GREEN_LED, LED_ACTIVE_HIGH ? LOW : HIGH);
      break;
    case STATE_IMPACT_DETECTED:
    case STATE_CONFIRMATION_WINDOW:
      digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? LOW : HIGH);
      digitalWrite(PIN_BUZZER, BUZZER_ACTIVE_HIGH ? LOW : HIGH);
      break;
    default:
      break;
  }
  
  // Enter new state
  currentState = newState;
  stateStartTime = millis();
  Serial.printf("[STATE] %d\n", currentState);
  
  // Entry actions
  switch (currentState) {
    case STATE_MONITORING:
      digitalWrite(PIN_GREEN_LED, LED_ACTIVE_HIGH ? HIGH : LOW);
      digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? LOW : HIGH);
      digitalWrite(PIN_BUZZER, BUZZER_ACTIVE_HIGH ? LOW : HIGH);
      break;
    case STATE_IMPACT_DETECTED:
      digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? HIGH : LOW);
      beepPattern(1);
      break;
    case STATE_CONFIRMATION_WINDOW:
      lastConfirmationBeep = 0;
      confirmationBeepState = false;
      break;
    case STATE_GPS_ACQUISITION:
      gpsRetryCount = 0;
      gpsFixAvailable = false;
      gpsLatitude = 0;
      gpsLongitude = 0;
      sim808Op = SIM808_GPS_ON;
      break;
    case STATE_SMS_SENDING:
      smsRetryCount = 0;
      sim808Op = SIM808_SMS_SEND;
      break;
    case STATE_EMERGENCY:
      beepPattern(3);
      break;
    case STATE_ERROR:
      break;
    default:
      break;
  }
}

void handleBoot() {
  Serial.println("[BOOT] Initializing hardware...");
  
  if (initMPU6050()) {
    Serial.println("[BOOT] MPU6050 OK");
  } else {
    Serial.println("[BOOT] MPU6050 FAILED");
    setState(STATE_ERROR);
    return;
  }
  
  calibrateBias();
  
  if (initSIM808()) {
    Serial.println("[BOOT] SIM808 OK");
  } else {
    Serial.println("[BOOT] SIM808 FAILED");
    setState(STATE_ERROR);
    return;
  }
  
  setState(STATE_SELF_TEST);
}

void handleSelfTest() {
  readMPU6050();
  if (accelData.magnitude > 0.5 && accelData.magnitude < 2.0) {
    Serial.println("[SELF_TEST] Sensors OK");
    setState(STATE_MONITORING);
  } else {
    Serial.println("[SELF_TEST] Sensor reading abnormal");
    setState(STATE_ERROR);
  }
}

void handleMonitoring() {
  unsigned long now = millis();
  
  if (now - lastSampleTime >= MPU6050_SAMPLE_MS) {
    lastSampleTime = now;
    readMPU6050();
    accelData.dynamicMagnitude = fabs(accelData.magnitude - 1.0);
    
    if (detectImpact()) {
      Serial.println("[MONITORING] IMPACT DETECTED!");
      setState(STATE_IMPACT_DETECTED);
    }
  }
}

void handleImpactDetected() {
  setState(STATE_CONFIRMATION_WINDOW);
}

void handleConfirmationWindow() {
  unsigned long now = millis();
  unsigned long elapsed = now - stateStartTime;
  
  // Flash red LED and beep every 500ms
  if (now - lastConfirmationBeep >= 500) {
    lastConfirmationBeep = now;
    confirmationBeepState = !confirmationBeepState;
    digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? (confirmationBeepState ? HIGH : LOW) : (confirmationBeepState ? LOW : HIGH));
    if (confirmationBeepState) {
      digitalWrite(PIN_BUZZER, BUZZER_ACTIVE_HIGH ? HIGH : LOW);
      delay(50);
      digitalWrite(PIN_BUZZER, BUZZER_ACTIVE_HIGH ? LOW : HIGH);
    }
  }
  
  // 15s timeout → GPS acquisition
  if (elapsed >= CONFIRMATION_WINDOW_MS) {
    Serial.println("[CONFIRMATION] Window expired, proceeding to emergency");
    setState(STATE_GPS_ACQUISITION);
  }
  
  // PHASE 2: Check for cancellation button/voice here
  // if (cancelPressed || voiceCancelDetected) {
  //   Serial.println("[CONFIRMATION] Cancelled by user");
  //   setState(STATE_MONITORING);
  // }
}

void handleGpsAcquisition() {
  unsigned long now = millis();
  unsigned long elapsed = now - stateStartTime;
  
  if (elapsed >= GPS_TIMEOUT_MS) {
    Serial.println("[GPS] Timeout, no fix");
    gpsFixAvailable = false;
    setState(STATE_SMS_SENDING);
    return;
  }
  
  if (sim808Op == SIM808_GPS_ON) {
    if (sendATCommand("AT+CGNSPWR=1", "OK", 2000)) {
      sim808Op = SIM808_GPS_WAIT;
      sim808LastCmdTime = now;
    }
  } else if (sim808Op == SIM808_GPS_WAIT) {
    if (now - sim808LastCmdTime >= 2000) {
      sim808Op = SIM808_GPS_READ;
    }
  } else if (sim808Op == SIM808_GPS_READ) {
    if (sendATCommand("AT+CGNSINF", "+CGNSINF:", 2000)) {
      if (gpsFixAvailable) {
        Serial.printf("[GPS] Fix acquired: %.6f, %.6f\n", gpsLatitude, gpsLongitude);
        setState(STATE_SMS_SENDING);
      } else if (gpsRetryCount < 10) {
        gpsRetryCount++;
        sim808Op = SIM808_GPS_WAIT;
        sim808LastCmdTime = now;
      } else {
        Serial.println("[GPS] Max retries, no fix");
        gpsFixAvailable = false;
        setState(STATE_SMS_SENDING);
      }
    }
  }
}

void handleSmsSending() {
  if (sim808Op == SIM808_SMS_SEND) {
    if (sendEmergencySMS(gpsLatitude, gpsLongitude, gpsFixAvailable)) {
      Serial.println("[SMS] Sent successfully");
      sim808Op = SIM808_IDLE;
      setState(STATE_EMERGENCY);
    } else {
      smsRetryCount++;
      if (smsRetryCount < SMS_RETRY_COUNT) {
        Serial.printf("[SMS] Failed, retry %d/%d\n", smsRetryCount, SMS_RETRY_COUNT);
        sim808Op = SIM808_SMS_SEND;
        delay(2000);
      } else {
        Serial.println("[SMS] Max retries exceeded");
        sim808Op = SIM808_IDLE;
        setState(STATE_EMERGENCY);  // Still go to emergency state
      }
    }
  }
}

void handleEmergency() {
  static unsigned long lastPattern = 0;
  if (millis() - lastPattern >= 5000) {
    lastPattern = millis();
    beepPattern(3);
  }
}

void handleError() {
  static unsigned long lastBlink = 0;
  if (millis() - lastBlink >= 200) {
    lastBlink = millis();
    static bool ledState = false;
    ledState = !ledState;
    digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? (ledState ? HIGH : LOW) : (ledState ? LOW : HIGH));
  }
}

// ========================
// MPU6050 FUNCTIONS
// ========================
bool initMPU6050() {
  // Wake up MPU6050
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x6B);  // PWR_MGMT_1
  Wire.write(0x00);  // Clear sleep mode
  if (Wire.endTransmission() != 0) return false;
  delay(100);
  
  // Set accelerometer full scale range (±16g)
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x1C);  // ACCEL_CONFIG
  Wire.write(MPU6050_ACCEL_FS << 3);
  Wire.endTransmission();
  
  // Set sample rate divider (100 Hz from 1 kHz DLPF)
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x19);  // SMPLRT_DIV
  Wire.write(9);     // 1kHz / (9+1) = 100Hz
  Wire.endTransmission();
  
  // Verify WHO_AM_I
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x75);  // WHO_AM_I
  Wire.endTransmission(false);
  Wire.requestFrom(MPU6050_ADDR, 1);
  if (Wire.available()) {
    uint8_t who = Wire.read();
    Serial.printf("[MPU6050] WHO_AM_I: 0x%02X\n", who);
    return who == 0x68;
  }
  return false;
}

void readMPU6050() {
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x3B);  // ACCEL_XOUT_H
  Wire.endTransmission(false);
  Wire.requestFrom(MPU6050_ADDR, 6);
  
  if (Wire.available() >= 6) {
    int16_t ax = (Wire.read() << 8) | Wire.read();
    int16_t ay = (Wire.read() << 8) | Wire.read();
    int16_t az = (Wire.read() << 8) | Wire.read();
    
    accelData.x = ax / MPU6050_ACCEL_LSB_G;
    accelData.y = ay / MPU6050_ACCEL_LSB_G;
    accelData.z = az / MPU6050_ACCEL_LSB_G;

    // Apply bias correction (set during calibration at boot)
    if (biasCalibrated) {
      accelData.x -= accelBias[0];
      accelData.y -= accelBias[1];
      accelData.z -= accelBias[2];
    }

    accelData.magnitude = sqrt(pow(accelData.x, 2) + pow(accelData.y, 2) + pow(accelData.z, 2));
    accelData.dynamicMagnitude = fabs(accelData.magnitude - 1.0);
  }
}

void calibrateBias() {
  Serial.println("[CALIB] Calibrating bias (keep device still)...");
  float sumX = 0, sumY = 0, sumZ = 0;
  int samples = 100;

  for (int i = 0; i < samples; i++) {
    readMPU6050();
    sumX += accelData.x;
    sumY += accelData.y;
    sumZ += accelData.z;
    delay(10);
  }

  accelBias[0] = sumX / samples;
  accelBias[1] = sumY / samples;
  accelBias[2] = sumZ / samples;
  biasCalibrated = true;

  Serial.printf("[CALIB] Bias: X=%.3f Y=%.3f Z=%.3f\n",
                 accelBias[0], accelBias[1], accelBias[2]);
}

bool detectImpact() {
  if (millis() - lastImpactTime < IMPACT_COOLDOWN_MS) {
    impactConsecutiveCount = 0;
    return false;
  }

  // dynamicMagnitude already computed in readMPU6050() with bias correction
  float dynamicMag = accelData.dynamicMagnitude;

  if (dynamicMag > IMPACT_THRESHOLD_G) {
    impactConsecutiveCount++;
    if (impactConsecutiveCount >= IMPACT_PERSISTENCE) {
      lastImpactTime = millis();
      impactConsecutiveCount = 0;
      return true;
    }
  } else {
    impactConsecutiveCount = 0;
  }
  return false;
}

// ========================
// SIM808 FUNCTIONS
// ========================
bool initSIM808() {
  Serial.println("[SIM808] Initializing...");
  
  if (!sendATCommand("AT", "OK", 1000)) return false;
  
  sendATCommand("ATE0", "OK", 1000);
  
  if (!sendATCommand("AT+CPIN?", "READY", 2000)) {
    Serial.println("[SIM808] SIM not ready");
    return false;
  }
  
  // Check network (0,1 or 0,5 = registered)
  bool regOK = sendATCommand("AT+CREG?", "+CREG: 0,1", 2000) ||
               sendATCommand("AT+CREG?", "+CREG: 0,5", 2000);
  if (!regOK) {
    Serial.println("[SIM808] Network not registered (may still work)");
  }
  
  sendATCommand("AT+CMGF=1", "OK", 1000);
  
  return true;
}

bool sendATCommand(const char* cmd, const char* expectedResponse, unsigned long timeout) {
  Serial.printf("[AT] >> %s\n", cmd);
  sim808Serial.println(cmd);
  
  unsigned long start = millis();
  String response = "";
  
  while (millis() - start < timeout) {
    while (sim808Serial.available()) {
      char c = sim808Serial.read();
      response += c;
      if (response.indexOf(expectedResponse) >= 0) {
        Serial.printf("[AT] << %s\n", response.c_str());
        return true;
      }
    }
    delay(1);
  }
  
  Serial.printf("[AT] << TIMEOUT: %s\n", response.c_str());
  return false;
}

String waitForResponse(unsigned long timeout) {
  unsigned long start = millis();
  String response = "";
  
  while (millis() - start < timeout) {
    while (sim808Serial.available()) {
      char c = sim808Serial.read();
      response += c;
    }
    delay(1);
  }
  
  if (response.length() > 0) {
    Serial.printf("[AT] << %s\n", response.c_str());
  }
  return response;
}

bool parseGPSInfo(String response) {
  // Parse +CGNSINF: 1,1,20230101020304.000,14.567890,121.123456,...
  int firstComma = response.indexOf(',');
  if (firstComma < 0) return false;
  
  int fieldIdx = 0;
  int lastComma = firstComma;
  float lat = 0, lon = 0;
  int fixStatus = 0;
  
  while (lastComma >= 0 && fieldIdx < 5) {
    int nextComma = response.indexOf(',', lastComma + 1);
    if (nextComma < 0) break;
    
    String field = response.substring(lastComma + 1, nextComma);
    
    if (fieldIdx == 0) {
      fixStatus = field.toInt();
    } else if (fieldIdx == 2) {
      lat = field.toFloat();
    } else if (fieldIdx == 3) {
      lon = field.toFloat();
    }
    
    lastComma = nextComma;
    fieldIdx++;
  }
  
  if (fixStatus == 1 && (lat != 0 || lon != 0)) {
    gpsLatitude = lat;
    gpsLongitude = lon;
    gpsFixAvailable = true;
    return true;
  }
  
  gpsFixAvailable = false;
  return false;
}

bool sendEmergencySMS(float lat, float lon, bool gpsAvail) {
  if (!sendATCommand("AT+CMGF=1", "OK", 1000)) return false;
  
  String cmd = "AT+CMGS=\"" + String(EMS_PHONE_NUMBER) + "\"";
  if (!sendATCommand(cmd.c_str(), ">", 2000)) return false;
  
  String message;
  if (gpsAvail) {
    message = "EMERGENCY ALERT\n\n";
    message += "Possible vehicle impact detected.\n\n";
    message += "Location:\n";
    message += "Latitude: " + String(lat, 6) + "\n";
    message += "Longitude: " + String(lon, 6) + "\n\n";
    message += "Google Maps:\n";
    message += "https://maps.google.com/?q=" + String(lat, 6) + "," + String(lon, 6) + "\n\n";
    message += "Please check the vehicle/occupant immediately.";
  } else {
    message = "EMERGENCY ALERT\n\n";
    message += "Possible vehicle impact detected.\n\n";
    message += "Location: GPS UNAVAILABLE (no fix)\n\n";
    message += "Please check the vehicle/occupant immediately.";
  }
  
  // Truncate message to fit SMS limit (160 chars max)
  if (message.length() > SMS_MAX_LENGTH) {
    message = message.substring(0, SMS_MAX_LENGTH - 3) + "...";
  }
  sim808Serial.print(message);
  sim808Serial.write(26);  // Ctrl+Z
  
  String response = waitForResponse(SMS_TIMEOUT_MS);
  return response.indexOf("+CMGS:") >= 0;
}

void processSIM808() {
  while (sim808Serial.available()) {
    char c = sim808Serial.read();
    sim808Buffer += c;
    
    if (c == '\n') {
      if (sim808Buffer.indexOf("+CGNSINF:") >= 0) {
        parseGPSInfo(sim808Buffer);
      }
      sim808Buffer = "";
    }
  }
}

// ========================
// PERIPHERAL FUNCTIONS
// ========================
void setGreenLED(bool on) {
  digitalWrite(PIN_GREEN_LED, LED_ACTIVE_HIGH ? (on ? HIGH : LOW) : (on ? LOW : HIGH));
}

void setRedLED(bool on) {
  digitalWrite(PIN_RED_LED, LED_ACTIVE_HIGH ? (on ? HIGH : LOW) : (on ? LOW : HIGH));
}

void setBuzzer(bool on) {
  digitalWrite(PIN_BUZZER, BUZZER_ACTIVE_HIGH ? (on ? HIGH : LOW) : (on ? LOW : HIGH));
}

void beepPattern(int pattern) {
  switch (pattern) {
    case 1:  // Single short beep
      setBuzzer(true); delay(100); setBuzzer(false);
      break;
    case 2:  // Double beep
      setBuzzer(true); delay(100); setBuzzer(false);
      delay(100);
      setBuzzer(true); delay(100); setBuzzer(false);
      break;
    case 3:  // Emergency pattern (long-short-long)
      setBuzzer(true); delay(300); setBuzzer(false);
      delay(100);
      setBuzzer(true); delay(100); setBuzzer(false);
      delay(100);
      setBuzzer(true); delay(300); setBuzzer(false);
      break;
  }
}

// ========================
// END OF FIRMWARE
// ========================

/*
  BUILD NOTES:
  1. Set EMS_PHONE_NUMBER in config.h before compiling
  2. Verify all GPIO pins match your ESP32 board
  3. Install ESP32 board package in Arduino IDE
  4. Select correct board (ESP32 Dev Module) and port
  5. Upload and test with serial monitor at 115200 baud
  
  TESTING:
  1. Power on, watch serial for initialization
  2. Green LED should turn on (MONITORING state)
  3. Tap/shake device to trigger impact detection
  4. Red LED should flash, buzzer beep for 15s
  5. After 15s, GPS acquisition starts
  6. SMS sent with location (or GPS unavailable)
  7. Emergency state: periodic beep pattern
*/
```

---

### 8.2 Build Instructions

1. **Arduino IDE:**
   - Install ESP32 board package (Espressif)
   - Select "ESP32 Dev Module"
   - Set upload speed 921600
   - Compile and upload

2. **PlatformIO:**
   ```ini
   [env:esp32dev]
   platform = espressif32
   board = esp32dev
   framework = arduino
   monitor_speed = 115200
   ```

3. **Before upload:**
   - Set `EMS_PHONE_NUMBER` in config.h
   - Verify GPIO assignments match your board
   - Install MPU6050 library (or use raw Wire.h)

---

### 8.3 Firmware Features Summary

- **State machine:** 9 states, non-blocking
- **Impact detection:** Dynamic magnitude with gravity removal, persistence check, cooldown
- **GPS acquisition:** AT+CGNSINF polling, 30s timeout
- **SMS sending:** Text mode, Ctrl+Z termination, retry mechanism
- **Peripherals:** Green/red LEDs, buzzer with configurable patterns
- **Fault tolerance:** Sensor failure → ERROR state, SIM808 timeout → ERROR state
- **Phase 2 ready:** GPIO4 reserved for push button, I²C available for voice module

---

### 8.4 Phase 2 Code Extensions

```cpp
// config.h additions:
#define PIN_CANCEL_BUTTON    4
#define BUTTON_DEBOUNCE_MS   50

// In handleConfirmationWindow():
if (digitalRead(PIN_CANCEL_BUTTON) == LOW) {
  if (millis() - lastButtonPress > BUTTON_DEBOUNCE_MS) {
    setState(STATE_MONITORING);
  }
}
```
