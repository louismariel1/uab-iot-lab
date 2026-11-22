# **SmartSecurePerimeter (SSP) — Consolidated IoT Lab Project Summary**

Consolidated baseline | 22 November 2026 | Prepared from the historical project messages, Deliverable 1 implementation report, SSP bootstrap and architectural clarification

## **1. Executive summary**

**SmartSecurePerimeter (SSP)** is an Internet of Things laboratory proof of concept developed around a Nordic nRF52840-DK running Zephyr RTOS and an Android Edge application written in Kotlin.

The project aims to demonstrate how physical sensor measurements can be acquired, processed and transformed into meaningful monitoring events and risk-based decisions.

The target architecture follows three layers:

**Device → Edge → Cloud**

The project has progressed beyond its initial BLE communication prototype. Its two principal achievements are:

- **BLE communication:** Advertising, connections, GATT services, notification transmission, Protocol v1 serialization, advertising recovery and reconnection have been demonstrated using nRF Connect on iPhone.
- **Sensor-ready firmware:** GPS, IMU, motion detection, location processing and sensor management abstractions have been implemented and integrated. A simulated motion value of 2100 mg has been observed in the firmware notification path.

The next major challenge is completing the Android Edge application and replacing simulated sensor inputs with real physical measurements.

**Overall assessment:** The communication foundation is established and the firmware architecture is sensor-ready, but the first complete physical sensor-driven SSP demonstration has not yet been established by the supplied evidence.

## **2. Project scope and objectives**

The SSP IoT Lab prototype is intended to demonstrate:

- Sensor acquisition and processing.
- Event-driven BLE communication.
- Edge computing and local monitoring decisions.
- State transitions based on detected events.
- Adaptive monitoring.
- Privacy-aware handling of information.
- Future Cloud integration and analytics.

The laboratory prototype must remain distinct from the broader SSP IoT Design Project. The design project describes the complete intended system, whereas this project implements and validates a practical subset.

## **3. System architecture**

**DEVICE — nRF52840-DK / Zephyr**

GPS and IMU acquisition → Location and Motion Processing → Sensor Manager → SSP Data Model → Event Generation → Protocol v1 → BLE GATT

BLE and simulated motion demonstrated; physical sensor acquisition pending

BLE notifications

**EDGE — Android / Kotlin**

BLE discovery → Connection → GATT subscription → Protocol decoding → Event display → State Engine → Adaptive Monitoring

BLE client implemented; full end-to-end evidence and state engine pending

HTTPS / MQTT — planned

**CLOUD — Future development**

Backend → Database → Dashboard → Analytics / Machine Learning

Not yet implemented in the supplied project baseline

## **4. Firmware architecture and implementation**

The most significant architectural improvement was replacing the original hardcoded motion value with a layered sensor-processing design.

| Layer                  | Components                         | Implementation                     |
| ---------------------- | ---------------------------------- | ---------------------------------- |
| Canonical data model   | `ssp_data_model.h`, `ssp_config.h` | Implemented                        |
| GPS abstraction        | `gps.h`, `gps.c`                   | Simulated                          |
| Location processing    | `location_engine.h`, `.c`          | Integrated                         |
| IMU abstraction        | `imu.h`, `imu.c`                   | Simulated                          |
| Motion processing      | `motion_detector.h`, `.c`          | Integrated                         |
| Sensor aggregation     | `sensor_manager.h`, `.c`           | Integrated                         |
| SSP event generation   | `ssp_event.c`                      | Integrated                         |
| Protocol serialization | `generated/ssp_protocol.h`         | Preserved                          |
| BLE transport          | `ssp_gatt.c`, `ssp_gatt.h`         | Preserved and previously validated |

The canonical data model includes `ssp_position_t`, `ssp_imu_sample_t`, `ssp_motion_t`, `ssp_event_type_t` and `ssp_event_data_t`.

### **Core architectural principle**

**Data Model → Protocol → Transport**

The SSP Data Model provides semantic separation from hardware-specific representations. The sensor abstractions and Sensor Manager provide implementation separation.

Consequently, a real GPS or IMU driver should be introduced without redesigning the motion-processing, event-generation or BLE transport layers. This is the intended architecture; compatibility must be confirmed through integration tests.

### **Firmware integration**

The historical implementation records report successful updates to:

- `CMakeLists.txt`, adding the new sensor and processing modules.
- `main.c`, initializing the Sensor Manager.
- `ssp_event.c`, replacing hardcoded motion data with sensor manager output.

A successful `west build` generated `zephyr.elf` and `zephyr.hex`.

The firmware subsequently produced simulated motion output and BLE notification logs, indicating that the new processing path was operational during the reported tests.

## **5. BLE communication and Protocol v1**

# **17**

Payload bytes

# **5s**

Test interval

# **0**

Reported TX errors

Protocol v1 uses a fixed 17-byte little-endian event representation.

| Byte offsets | Field          | Encoding  |
| ------------ | -------------- | --------- |
| 0            | Version        | uint8     |
| 1            | Event type     | uint8     |
| 2            | Status         | uint8     |
| 3–6          | Device ID      | uint32 LE |
| 7–10         | Sequence       | uint32 LE |
| 11–14        | Timestamp (ms) | uint32 LE |
| 15–16        | Motion (mg)    | uint16 LE |

The implementation report records a captured payload containing the simulated motion value `0x0834`, corresponding to 2100 mg.

### **Validated BLE functionality**

The recorded tests demonstrate advertising, discovery, connection, GATT discovery, notification subscription, transmission, disconnection detection, automatic advertising restart and reconnection.

The report also records four transmissions at five-second intervals with zero reported transmission errors.

Two validation limitations remain important: explicit CCCD processing evidence and complete Android-side reception, decoding and event display were not established in that report.

Therefore, **M3 is technically validated for the tested BLE communication path**, while complete Android Edge validation remains a separate requirement.

## **6. Hardware and development environment**

| Component          | Baseline                      |
| ------------------ | ----------------------------- |
| Development board  | Nordic nRF52840-DK            |
| Firmware platform  | Zephyr RTOS 4.5.0-rc1         |
| Firmware language  | C                             |
| Edge platform      | Android / Kotlin              |
| Firmware workspace | `/home/louis/zephyrproject`   |
| SSP repository     | `/home/louis/ssp-iot-poc`     |
| Python environment | Python 3.12.15, `zephyr-3.12` |
| Build and flashing | Zephyr `west`                 |
| Diagnostics        | UART serial monitor           |
| BLE test client    | nRF Connect on iPhone         |
| Version control    | Git / GitHub                  |

The historical hardware inventory identifies the nRF52840-DK, Android smartphone and GPS module as available, while the physical IMU was awaiting delivery. Current availability should be reconfirmed before the next implementation phase.

## **7. Consolidated milestone status**

Statuses reflect the strongest evidence provided, rather than simply copying historical completion labels.

| Milestone                     | Consolidated status                                      |
| ----------------------------- | -------------------------------------------------------- |
| M0 — Development environment  | Complete                                                 |
| M1 — SSP specification        | Complete                                                 |
| M2 — SSP event generation     | Complete with simulated data                             |
| M3 — Device-to-Edge BLE       | Validated with iPhone client                             |
| Sensor-ready architecture     | Implemented and integrated                               |
| Sensor firmware build         | Successful                                               |
| Simulated motion runtime      | Demonstrated                                             |
| Deliverable 1                 | Final report exists; evidence package needs confirmation |
| M4 — Android Edge             | Partially implemented                                    |
| M5 — Physical IMU integration | Pending                                                  |
| M6 — SSP state engine         | Planned                                                  |
| M7 — Adaptive monitoring      | Planned                                                  |
| M8 — Location event model     | Planned                                                  |
| M9 — Physical GPS integration | Pending                                                  |
| M10+ — Cloud and analytics    | Future                                                   |

A historical bootstrap places M4 as the active milestone, while subsequent work also progressed on sensor architecture. These workstreams can continue in parallel, but should retain separate acceptance criteria.

## **8. Deliverable 1 status**

The Device-to-Edge BLE Communication Validation report is drafted and documents the communication architecture, BLE design, Protocol v1, regression tests, event-generation pipeline and limitations.

The supporting evidence package should contain BLE discovery and connection screenshots, notification subscription evidence, UART connection/recovery logs, notification logs, captured payloads and a repeatable demonstration procedure.

The report is marked as a final version, but that does not by itself prove formal submission or that every supporting artifact is complete.

**Deliverable 1 should therefore be treated as technically mature, with final evidence and submission status to verify.**

## **9. First complete SSP vertical slice**

The primary functional demonstration remains:

Physical movement

Real IMU X/Y/Z acquisition

Motion detection and threshold evaluation

MOTION_EVENT generation

Protocol v1 serialization

BLE notification

Android reception and decoding

NORMAL → ELEVATED

**Visible monitoring state**

The acceptance criteria are that actual movement produces a sensor measurement, the firmware generates an SSP event without synthetic input, the event reaches Android and the Edge application automatically changes its displayed monitoring state.

This demonstration combines M4, M5 and M6 into a single functional validation scenario without treating them as the same milestone.

## **10. Outstanding implementation gaps**

| Area                | Missing capability                                             |
| ------------------- | -------------------------------------------------------------- |
| Android             | Verified complete reception, decoding and event display        |
| State engine        | NORMAL → ELEVATED based on a valid motion event                |
| IMU                 | Real I²C acquisition and motion validation                     |
| GPS                 | UART acquisition and NMEA parsing                              |
| Location            | Real fix validation and SSP position population                |
| Protocol            | Future location/geofence event representation                  |
| Adaptive monitoring | State-dependent notification policy                            |
| Stability           | Extended regression and intermittent kernel-oops investigation |
| Cloud               | Backend, persistence, dashboard and analytics                  |

The existing Protocol v1 format must not be assumed capable of transmitting full GPS coordinates. Location transmission requires an explicitly designed and validated extension.

## **11. Recommended priorities for the next phase**

The consolidated information supports this sequence:

1. **Verify the current repository and firmware baseline.**

   Confirm the actual source revision, build configuration, hardware status and most recent runtime evidence.
2. **Close Deliverable 1.**

   Verify evidence completeness, final report consistency and submission status.
3. **Complete Android Edge (M4).**

   Demonstrate packet reception, Protocol v1 decoding and event presentation.
4. **Implement the minimal state engine (M6).**

   Validate NORMAL → ELEVATED using the existing simulated motion events before requiring physical hardware.
5. **Integrate physical sensors.**

   Implement real GPS UART/NMEA processing and IMU I²C acquisition, prioritizing tasks according to hardware availability.
6. **Demonstrate the first real vertical slice.**

   Prove the complete physical movement → Device → BLE → Android → monitoring-state chain.

Cloud, database, dashboard, analytics and ML should remain later milestones until the core Device-to-Edge functionality is stable.

## **12. Project conclusions and baseline for the next bootstrap**

The historical records establish that SSP has completed its foundational BLE communication work and successfully integrated a modular sensor-processing architecture.

The key engineering achievement is the separation of physical sensor acquisition, processing, canonical event representation and BLE transport. This allows the team to introduce real sensors while preserving the validated communication infrastructure.

The most important unresolved work is completing Android Edge validation, implementing state transitions and demonstrating that real sensor measurements—not simulated values—drive SSP monitoring behavior.

**This consolidated summary should serve as the reference for the next project bootstrap.** The bootstrap can then be shorter and more operational, containing the verified current baseline, repository structure, milestone status, immediate tasks, acceptance criteria and commands needed to resume development without repeating completed work.

The next step is to reconcile this historical baseline against the current repository and latest hardware/test evidence before finalizing that bootstrap.
