# IoT-Based Vehicle Impact Detection System
## Complete Documentation

**Project:** Development of an IoT-Based Vehicle Impact Detection System with GPS Location Tracking and Automated EMS Notification at Taytay, Rizal

**Phase:** Phase 1 — Current Prototype (impact detection → countdown → GPS → SMS)

---

## Documentation Index

| # | File | Purpose |
|---|------|---------|
| 1 | `SETUP.md` | Project setup, requirements, assembly, enclosure, chapter 1–3 guide |
| 2 | `BOM.md` | Bill of materials with Lazada links |
| 3 | `SYSTEM-ARCHITECTURE.md` | High-level system design, data flow, power architecture |
| 4 | `BLOCK-DIAGRAM.md` | System block diagrams |
| 5 | `FLOWCHART.md` | System flowchart, state machine |
| 6 | `WIRING.md` | Pin assignments, complete wiring guide |
| 7 | `STACKS.md` | Software stack, firmware architecture, sensor/GPS/SMS guides |
| 8 | `FIRMWARE.md` | Complete ESP32 firmware code |
| 9 | `TESTING.md` | Testing, calibration, validation, checklist |
| 10 | `TROUBLESHOOTING.md` | Common failures & fixes |

---

## Quick Reference: System Flow

```
POWER ON → INIT → CHECK MPU6050 → CHECK SIM808 → NORMAL MONITORING (GREEN LED)
  → MPU6050 loop
    → IMPACT DETECTED → RED LED ON → BUZZER WARNING → 15 s CONFIRMATION WINDOW
      → CANCELLED? → Reset to monitoring (Phase 2 only)
      → NOT CANCELLED → GPS ACQUISITION → FORMAT SMS → SEND SMS → EMERGENCY STATE
```

---

## Phase 1 vs Phase 2

**Phase 1 (Implemented):** Impact detection → warning → countdown → GPS → SMS

**Phase 2 (Deferred):** Push-button cancellation, voice recognition/cancellation

---

## Critical Anti-Hallucination Rules

- No invented GPIO pins — all marked TBD until verified against actual board
- No invented component models — generic where exact part unverified
- No assumed 5 V tolerance — verify before connecting
- No cloud/EMS integration claimed — SMS only via SIM808
- No voice/push-button in Phase 1

---

## Safety Notes

- Li-Po batteries: verify matched pairs before paralleling
- Boost converter: measure output before connecting ESP32
- SIM808: transmit current bursts up to ~2 A — ensure power path can handle it
- SIM808: install 1000µF 25V capacitor at power input to smooth voltage drops during TX bursts
- TP4056: charger only, NOT a 5 V regulator
