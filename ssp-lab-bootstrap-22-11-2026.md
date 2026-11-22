# **SSP IoT Lab — Final Project Bootstrap**

SmartSecurePerimeter (SSP) | Internet of Things Laboratory | Baseline: 22 November 2026

**Document purpose:** Establish an authoritative working baseline for the SSP laboratory prototype, preserve completed engineering work, distinguish validated capabilities from planned functionality, and define the next executable development steps.

**Project:** SmartSecurePerimeter (SSP)\
**Device:** Nordic nRF52840-DK\
**Firmware:** Zephyr RTOS / C\
**Edge:** Android / Kotlin\
**Independent BLE validation:** nRF Connect on iPhone\
**Current status:** BLE communication demonstrated; sensor-ready firmware architecture implemented; Android end-to-end validation and physical sensor acquisition pending.

This bootstrap concerns the **SSP IoT Laboratory prototype**, not the separate full-system SSP IoT Design Project.

## **1. Project Vision**

SmartSecurePerimeter is an event-driven IoT security and monitoring prototype designed to demonstrate distributed intelligence, Edge computing, adaptive monitoring, privacy-aware sensing and risk-based decision-making.

The central objective is to transform physical measurements into meaningful SSP events rather than continuously streaming raw sensor data.

The target processing chain is:

Physical Sensors

Sensor Acquisition

Local Processing

**SSP Event Generation**

BLE Communication

Android Edge

State & Risk Assessment

Adaptive Monitoring

Cloud Services (Future)

The first complete functional demonstration will show real physical movement producing an SSP motion event that reaches Android and causes a visible monitoring-state transition.

## **2. Architectural Principles**

The SSP architecture separates hardware acquisition, sensor processing, event representation and communication.

| Responsibility              | Implementation                           |
| --------------------------- | ---------------------------------------- |
| Hardware acquisition        | UART / I²C drivers                       |
| Sensor abstraction          | `gps.c`, `imu.c`                         |
| Sensor processing           | `location_engine.c`, `motion_detector.c` |
| Sensor integration          | `sensor_manager.c`                       |
| Canonical information model | `ssp_data_model.h`                       |
| Event generation            | `ssp_event.c`                            |
| Protocol serialization      | `generated/ssp_protocol.h`               |
| BLE transport               | `ssp_gatt.c`                             |

The governing principle is:

**SSP Data Model → Protocol → Transport**

The canonical data model provides semantic independence from the hardware, while sensor abstractions and the Sensor Manager provide implementation separation.

Real sensor integration should preserve the higher-level interfaces wherever technically feasible. Hardware configuration, driver dependencies and sensor-validity handling may still require additional changes.

## **3. Current System Architecture**

**Device — nRF52840-DK / Zephyr**

GPS Abstraction

`gps.clocation_engine.c`Simulated

IMU Abstraction

`imu.cmotion_detector.c`Simulated

**Sensor Manager → SSP Data Model → Event System → Protocol v1 → BLE GATT**

BLE notifications

**Edge — Android / Kotlin**

BLE discovery → Connection → GATT subscription → Protocol decoding → Event display → State Engine

Client infrastructure implemented; complete application validation pending

HTTPS / MQTT — future

**Cloud — Future**

Backend, database, dashboard, analytics and machine learning

The Device and Edge layers are the immediate implementation scope. Cloud services remain future work.

## **4. Development Environment**

The recorded development environment is:

| Item                | Configuration               |
| ------------------- | --------------------------- |
| Host                | Windows 11 with WSL Ubuntu  |
| Zephyr              | 4.5.0-rc1                   |
| West                | 1.5.0                       |
| Python              | 3.12.15                     |
| pyenv environment   | `zephyr-3.12`               |
| Firmware workspace  | `/home/louis/zephyrproject` |
| Project repository  | `/home/louis/ssp-iot-poc`   |
| Android development | Android Studio / Kotlin     |
| BLE validation      | nRF Connect on iPhone       |

The primary firmware project is located at:

```
device/firmware/ble_peripheral/
```

Important supporting paths include:

```
docs/protocol/ssp-ble-v1.md
docs/testing/ssp-ble-integration.md
docs/testing/ssp-ble-test-results.md

tools/rebuild_ssp.sh
tools/compile_and_flash_test.sh
tools/flash_and_uart_smoke_test.sh

edge/android/
```

These paths reflect the historical project records and must be confirmed against the current repository before executing scripts.

## **5. Major Engineering Challenges Solved**

### **Zephyr workspace and repository separation**

The SSP repository is located outside the Zephyr workspace. Earlier issues occurred when `west` was installed but its workspace could not be resolved from the current directory.

Build automation was introduced to locate the correct workspace and validate the toolchain.

### **Build automation**

The project records identify `tools/rebuild_ssp.sh` as the build automation entry point, with environment activation, workspace checks, tool validation and artifact verification.

Other scripts support compilation, flashing and UART smoke testing.

### **BLE advertising recovery**

An earlier problem caused unreliable rediscovery following disconnect.

The firmware was modified to schedule a delayed advertising restart.

Subsequent testing demonstrated advertising recovery, rediscovery and reconnection, including successful restart logs.

### **Sensor architecture refactoring**

The previous hardcoded motion source was replaced with an integrated simulated sensor-processing pipeline.

The firmware was rebuilt successfully, and subsequent runtime evidence included a simulated motion value of 2100 mg transmitted through BLE.

## **6. SSP Data Model**

The canonical SSP data model is defined in:

```
device/firmware/ble_peripheral/src/ssp_data_model.h
```

Core types include:

```
ssp_event_data_t
ssp_position_t
ssp_motion_t
ssp_imu_sample_t
ssp_event_type_t
```

The model represents what SSP knows about events, position and motion independently of the physical sensors that generated the measurements.

**Validated status:**

- Data model implemented.
- Data model integrated into firmware.
- Simulated motion-event pathway demonstrated on hardware.
- Real position fields and physical IMU measurements not yet validated.

The existence of a data structure is not treated as proof that every field has been exercised with real sensor data.

## **7. Protocol v1**

SSP BLE Protocol v1 uses a fixed 17-byte payload with little-endian encoding for multibyte fields.

| Byte offsets | Field            | Type   |
| ------------ | ---------------- | ------ |
| 0            | Protocol version | uint8  |
| 1            | Event type       | uint8  |
| 2            | Event status     | uint8  |
| 3–6          | Device ID        | uint32 |
| 7–10         | Sequence         | uint32 |
| 11–14        | Timestamp (ms)   | uint32 |
| 15–16        | Motion (mg)      | uint16 |

The current implementation supports motion-event transmission. Future event types may include `LOCATION_EVENT`, `GEOFENCE_EVENT` and `TAMPER_EVENT`.

The existing 17-byte representation does not contain coordinates, altitude or satellite information.

**Protocol decision:** Preserve Protocol v1 for the existing motion-event path. Any future location payload or new event representation must be explicitly specified, versioned or otherwise compatibility-managed, and tested before deployment.

The event-status byte retains its current semantics, including the recorded definitions:

```
#define SSP_STATUS_OK           0
#define SSP_STATUS_SENSOR_ERROR 1
```

Monitoring states will initially be managed within Android rather than by redefining this field.

## **8. Current Sensor Architecture**

The sensor architecture is implemented and integrated into the firmware, but acquisition remains simulated.

| Module              | Current capability                             | Pending implementation             |
| ------------------- | ---------------------------------------------- | ---------------------------------- |
| `gps.c`             | GPS abstraction and simulated position         | UART acquisition                   |
| `location_engine.c` | Position-processing abstraction                | Real GPS fix processing            |
| `imu.c`             | IMU abstraction and simulated samples          | I²C acquisition                    |
| `motion_detector.c` | Motion metric and threshold evaluation         | Validation using real acceleration |
| `sensor_manager.c`  | Sensor coordination and event data integration | Real sensor integration            |

### **GPS processing**

Current:

`Simulated GPS → GPS Abstraction → Location Engine → SSP Position`

Target:

`Physical GPS → UART → NMEA Parser → GPS Abstraction → Location Engine → SSP Position`

The real GPS implementation must validate fix status, latitude, longitude and available satellite information.

### **IMU processing**

Current:

`Simulated IMU → Motion Detector → Sensor Manager → MOTION_EVENT`

Target:

`Physical IMU → I²C → X/Y/Z Measurements → Motion Detector → Sensor Manager → MOTION_EVENT`

The existing motion-processing interface should be preserved wherever feasible.

### **Architectural constraints**

Real sensor integration must not introduce unnecessary changes to the existing event system or BLE transport.

Additional Zephyr configuration, devicetree changes, hardware drivers and sensor-validity handling may be necessary.

The existing motion-event protocol should remain unchanged during physical IMU integration.

## **9. BLE Implementation Status**

The reported BLE tests used nRF Connect on iPhone as an independent validation client.

| Test                             | Reported result            |
| -------------------------------- | -------------------------- |
| BLE advertising                  | PASS                       |
| Device discovery                 | PASS                       |
| Connection                       | PASS                       |
| GATT service discovery           | PASS                       |
| Notification subscription        | PASS                       |
| Notification transmission        | PASS                       |
| Disconnection handling           | PASS                       |
| Advertising recovery             | PASS                       |
| Reconnection                     | PASS                       |
| Protocol payload serialization   | PASS                       |
| Sequence progression             | PASS                       |
| Periodic heartbeat               | PASS                       |
| Android end-to-end event display | Not yet formally validated |

### **Recorded runtime evidence**

The firmware produced logs including:

```
[CONNECT] CONNECTED count=2
[MONITOR] Notification session started
[MOTION] detected=1 motion=2100
[NOTIFY] #14 len=17
[NOTIFY] #15 len=17
[NOTIFY] #16 len=17
[NOTIFY] #17 len=17
```

The recorded notification session transmitted four notifications over approximately 21 seconds, with zero reported transmission errors.

The 2100 mg motion value originated from simulated IMU inputs.

### **Validation boundary**

These results demonstrate the Device-to-BLE-client communication path.

They do not yet establish the complete Android application path:

`Device → BLE → Android → Decode → Display`

That remains the primary objective of M4.

## **10. Current Milestone Status**

The following status table is the authoritative working baseline until superseded by repository inspection or new runtime evidence.

| Milestone / Capability        | Status                                     |
| ----------------------------- | ------------------------------------------ |
| M0 — Development Environment  | Complete                                   |
| M1 — SSP Specification        | Complete                                   |
| M2 — SSP Event Generation     | Complete, simulated                        |
| M3 — BLE Communication        | Demonstrated with iPhone                   |
| Deliverable 1                 | Report prepared; evidence to finalize      |
| Canonical SSP Data Model      | Implemented and integrated                 |
| GPS Software Architecture     | Implemented, simulated                     |
| IMU Software Architecture     | Implemented, simulated                     |
| Motion Processing             | Simulated runtime demonstrated             |
| Sensor Manager                | Integrated; simulated runtime demonstrated |
| Firmware Build                | Successful                                 |
| Firmware Flash and Execution  | Demonstrated; revision to confirm          |
| BLE Regression                | Passed recorded iPhone tests               |
| M4 — Android Edge             | In progress                                |
| M5 — Real IMU Acquisition     | Pending hardware confirmation              |
| M6 — State Engine             | Planned                                    |
| M7 — Adaptive Monitoring      | Planned                                    |
| M8 — Location Event Model     | Planned                                    |
| M9 — Real GPS Acquisition     | Planned                                    |
| M10+ — Cloud / Analytics / ML | Future                                     |

**Status terminology:**

- *Implemented:* Code or functionality exists.
- *Demonstrated:* Observed working in a reported test.
- *Validated:* Tested against explicit acceptance criteria.
- *Planned:* Not yet established as implemented.

A successful build does not establish runtime correctness, and simulated validation does not establish physical sensor correctness.

## **11. Future SSP State Model**

The Edge application will maintain the initial monitoring states:

**NORMAL**

Baseline monitoring

**ELEVATED**

Motion event detected

**CRITICAL**

Future risk assessment

### **Initial state-engine behavior**

For the first implementation:

1. Android begins in `NORMAL`.
2. A valid `MOTION_EVENT` is received and decoded.
3. Android evaluates the event.
4. The monitoring state changes to `ELEVATED`.
5. The new state becomes visible in the UI.

The `CRITICAL` state, reset conditions, event expiry and more advanced risk rules will be specified separately.

### **Protocol decision**

Protocol v1 byte 2 remains an **event-status field**, not an Android monitoring-state field.

The Android state engine operates independently of this byte.

This allows the initial state engine to be implemented without changing the 17-byte payload or breaking the validated communication path.

### **Adaptive monitoring**

A future milestone may introduce state-dependent monitoring intervals, such as the previously proposed 10-second NORMAL and 2-second ELEVATED intervals.

These values are design targets, not validated operating parameters. Any change in Device notification frequency requires an explicit Device–Edge control mechanism and associated tests.

## **12. Immediate Next Steps**

Development will proceed through three stages, beginning with verification of the existing implementation.

### **Stage 1 — Consolidate and validate**

Objective: Establish a reproducible, evidence-backed firmware and BLE baseline.

**Stage 1 checklist**

0/6 completed

Confirm repository revision

Record Git branch, commit and working-tree status

Inspect firmware integration

Verify sensor modules, event path and build configuration

Rebuild firmware

Confirm successful compilation and expected artifacts

Flash and smoke test

Verify startup, heartbeat and simulated motion processing

Run BLE regression

Verify advertising, subscription, notifications and reconnect

Finalize Deliverable 1

Reconcile report, screenshots, logs and captured payloads

### **Stage 2 — Complete Android Edge**

Objective: Demonstrate a complete Device-to-Android event path and an initial monitoring-state transition.

**Stage 2 checklist**

0/6 completed

BLE discovery and connection

Android finds and connects to SSP-POC

GATT subscription

Notifications are enabled and received

Protocol decoding

Validate version, length, fields and little-endian values

Event presentation

Show event type, sequence, timestamp and motion value

Basic state engine

Valid MOTION_EVENT triggers NORMAL → ELEVATED

End-to-end evidence

Capture screenshots, Android logs and matching UART events

### **Stage 3 — Integrate physical sensors**

Objective: Replace simulated inputs with real measurements and complete the first physical SSP vertical slice.

**Stage 3 checklist**

0/7 completed

Confirm hardware inventory

Identify GPS and IMU models, voltage, pins and interfaces

GPS UART acquisition

Receive raw NMEA sentences

GPS parsing

Extract and validate real fixes and coordinates

SSP position validation

Populate canonical position data from GPS

IMU I²C acquisition

Read real X/Y/Z acceleration samples

Physical motion detection

Generate MOTION_EVENT from real movement

Complete vertical slice

Demonstrate physical movement → Android ELEVATED state

GPS and IMU integration may be developed in parallel after Stage 2, depending on hardware availability. GPS acquisition alone does not require immediate changes to the existing BLE payload.

## **13. First Complete SSP Vertical Slice**

The primary system-level acceptance test is:

`Physical Movement → Real IMU → Motion Detection → MOTION_EVENT → BLE → Android → State Engine → NORMAL → ELEVATED → Visible User State`

The demonstration passes when all of the following are established:

| Criterion          | Required evidence                            |
| ------------------ | -------------------------------------------- |
| Physical movement  | Real IMU readings change with movement       |
| Sensor acquisition | Hardware driver returns valid measurements   |
| Motion detection   | Threshold evaluation produces a motion event |
| Event generation   | No synthetic motion values are used          |
| BLE transmission   | Valid Protocol v1 event received             |
| Android decoding   | Event fields match firmware output           |
| State transition   | Android automatically enters ELEVATED        |
| User feedback      | Updated monitoring state visible on screen   |
| Repeatability      | Demonstration succeeds across repeated runs  |

The first complete vertical slice is **not yet validated** according to the historical evidence.

It is the main integration milestone following completion of the Android Edge functionality and physical IMU acquisition.

## **14. Project Position**

The SSP IoT Laboratory project has progressed from a basic BLE communication prototype to a **BLE-validated, sensor-ready event-driven firmware platform**.

The established foundations are:

- BLE advertising, GATT communication, notification transmission, advertising recovery and reconnection.
- Protocol v1 serialization and captured payload validation.
- Canonical SSP Data Model.
- GPS and IMU software abstractions.
- Motion detection and Sensor Manager integration.
- Successful firmware builds and reported hardware execution using simulated motion.

The principal remaining objectives are:

- Finalize Deliverable 1 evidence.
- Validate the complete Android Edge reception and display path.
- Implement the Android monitoring-state engine.
- Integrate physical GPS and IMU acquisition.
- Demonstrate the first complete sensor-driven SSP vertical slice.

**Project execution rule:** Preserve the validated architecture and completed functionality. Modify existing components only when necessary for hardware integration, compatibility, correctness or explicit acceptance criteria.

## **Development resumption — First executable action**

Before changing firmware, establish the current repository baseline.

From WSL Ubuntu, run:

```
cd /home/louis/ssp-iot-poc

printf '\n=== Repository ===\n'
pwd
git status --short
git branch --show-current
git log -1 --format='%h %cs %s'

printf '\n=== Firmware modules ===\n'
find device/firmware/ble_peripheral \
  -maxdepth 3 -type f | sort

printf '\n=== Tool scripts ===\n'
find tools -maxdepth 1 -type f | sort

printf '\n=== Android project ===\n'
find edge/android -maxdepth 2 -type f | head -40
```

This first command set is read-only. It establishes the repository revision and available source files without modifying, rebuilding or flashing anything.

**Next action:** Review the command output, identify any differences from this bootstrap, then select the exact firmware build and BLE regression procedure already implemented in the repository.
