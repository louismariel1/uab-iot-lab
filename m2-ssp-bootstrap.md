# SmartSecurePerimeter (SSP) — Project Bootstrap

Document: SSP_PROJECT_BOOTSTRAP.md
Purpose: Single source of truth for project context, current status, architecture, milestones, and implementation strategy.
Status: Reconstructed after loss of the previous ChatGPT session.
Current milestone: M2 — Sensor Integration.
Immediate engineering objective: Reach a working Device → BLE → Android Edge demonstration as quickly as possible, using batch-oriented code generation and automated verification.

1. Project objective

SmartSecurePerimeter (SSP) is an IoT proof of concept demonstrating distributed device-to-edge intelligence.

A physical device based on the Nordic nRF52840 detects movement using an accelerometer/IMU, processes the measurement locally, transmits a compact event over Bluetooth Low Energy (BLE), and allows an Android application to validate the event and update its monitoring state.

The initial deterministic state transition is:

Physical movement
       |
       v
Accelerometer / IMU
       |
       v
nRF52840 + Zephyr firmware
       |
       v
Local motion processing
       |
       v
SSP MOTION_EVENT
       |
       v
BLE GATT notification
       |
       v
Android Edge application
       |
       v
Event validation
       |
       v
NORMAL -> ELEVATED


The first objective is a repeatable, measurable vertical slice, not a production-ready IoT platform.

Machine learning, cloud integration, GNSS positioning, geofencing, advanced risk modelling, and production security are deferred.

2. Technology stack
Device
Hardware: Nordic nRF52840-DK.
RTOS: Zephyr.
Language: C.
Development and build: west, CMake, Zephyr configuration.
Debugging and diagnostics: UART, board flashing, build/test scripts.
Sensor: accelerometer/IMU model not yet confirmed in the recovered information.
Communication: Bluetooth Low Energy peripheral, GATT service and notifications.
Edge
Platform: Android.
Language: Kotlin.
Communication: BLE scanning, connection, service discovery and notifications.
Responsibilities: payload validation, sequence/timestamp handling, event interpretation, state transitions, and local display.
Cloud

Cloud services and databases are outside the first vertical slice.

3. Project roadmap

The original roadmap has 21 phases, numbered 0–20, consolidated into 11 proposed GitHub milestones.

Milestone	Scope	Completion criterion
M0	Development environment	Reproducible firmware, Android and backend environments
M1	SSP requirements	Use case, architecture, events, state model and KPIs defined
M2	Sensor integration	Reliable, timestamped measurements on the nRF52840
M3	Device firmware and BLE protocol	Data model and protocol specified; device transmits valid SSP data
M4	Android BLE integration	Android receives and displays live device data
M5	Edge processing and risk	Validation, processing and risk classification work
M6	Adaptive monitoring and Edge ML	Adaptive policies and ML are implemented and evaluated
M7	Cloud backend and database	Edge data is transmitted and stored
M8	Dashboard and alerts	State, events, history and alerts are displayed
M9	End-to-end validation and KPIs	Security, privacy, performance and failure scenarios are evaluated
M10	Final integration and documentation	Clean checkout supports reproducible deployment and demonstration

The boundaries are provisional. In particular, M3/M4 may be adjusted to reflect the actual firmware and Android integration sequence.

4. Recovered status and evidence
M1 — Requirements

The recovered specification defines:

The physical movement scenario.
Device-to-Edge architecture.
A logical SSP event model.
Initial NORMAL and ELEVATED states.
A first vertical-slice acceptance test.
Initial performance and reliability KPIs.
Explicitly deferred functionality.

M1 is documented as complete in the recovered project notes.

Development environment

The previous notes report that:

The nRF52840 firmware was flashed successfully.
A UART listener received SSP PoC Hello World! nRF52840.
The automated test passed after its expected UART string was corrected.

This establishes evidence for the reported basic flash/UART path. The actual repository and logs must be inspected before treating this as independently reverified.

M2 — Sensor Integration

M2 is the current milestone.

The original implementation plan proposed:

A sensor-independent acquisition interface.
A synthetic sensor.
A logical measurement data structure.
Sequence numbering and Zephyr uptime timestamps.
Acceleration magnitude calculation.
Deterministic motion detection.
UART diagnostics.
Build and flash validation.

The supplied implementation was proposed, but its files, build, flash and runtime behavior have not been verified from the recovered conversation.

Current status: M2 is not confirmed complete.

5. Initial SSP data model

The logical measurement should support:

device_id
sequence
timestamp_ms
Acceleration X/Y/Z
Derived motion metric
Event type
Device status

Initial event types:

NONE
MOTION_EVENT


Initial device statuses:

OK
SENSOR_ERROR


Initial Edge states:

NORMAL
ELEVATED


The logical model must remain independent of the physical sensor driver and BLE byte encoding.

Use Zephyr uptime initially for relative timestamps. Absolute time synchronization is a separate future decision.

6. Motion-processing design

The proposed initial processing pipeline is:

Sensor sample
    |
    v
Input validation
    |
    v
Acceleration X/Y/Z
    |
    v
Motion metric
    |
    v
Configurable deterministic threshold
    |
    v
NONE or MOTION_EVENT


A preliminary implementation uses acceleration magnitude:

sqrt(x*x + y*y + z*z)

The recovered example uses a threshold of 1.20, but this is experimental and must not be treated as a validated physical-motion threshold. The sensor's units, gravity handling, sampling rate, filtering and calibration must be established before physical detection performance is claimed.

Synthetic data is appropriate for validating the software pipeline independently of sensor availability.

7. Proposed firmware architecture

Existing baseline application:

device/firmware/hello_world/
├── CMakeLists.txt
├── prj.conf
└── src/
    └── main.c


Preserve this known baseline.

Proposed separate SSP application:

device/firmware/ssp_ble_demo/
├── CMakeLists.txt
├── prj.conf
└── src/
    ├── main.c
    ├── ssp/
    │   ├── ssp_types.h
    │   └── ssp_motion.c
    ├── sensor/
    │   ├── sensor_interface.h
    │   └── sensor_sim.c
    └── ble/
        ├── ssp_ble.h
        └── ssp_ble.c


This structure is provisional and must be reconciled with the real repository before code generation.

Responsibilities:

main.c: initialization and application orchestration.
ssp_types.h: logical data types and event definitions.
ssp_motion.c: motion metric and event detection.
sensor_interface.h: sensor abstraction.
sensor_sim.c: repeatable synthetic samples.
ssp_ble.h and ssp_ble.c: BLE service, advertising, connection and notification behavior.

The physical IMU driver should later replace the simulator behind the same interface.

GNSS is not part of the immediate implementation.

8. BLE and protocol design

The first BLE implementation shall provide:

Peripheral advertising.
An SSP-specific GATT service.
Device identification/status where appropriate.
An event characteristic.
Notification capability.
Connection and subscription handling.
A documented payload format.

UUIDs, characteristic properties, payload encoding, field widths, units, versioning and validation rules must be agreed before declaring the protocol stable.

A short textual payload was suggested for the first demonstration:

SSP,1,42,123456


This is only an illustrative proposal, not a finalized protocol. The final representation should be chosen deliberately, documented, and tested against matching device and Android implementations.

Do not independently invent incompatible packet formats in different code-generation batches.

9. Android Edge application

The initial Kotlin application should remain minimal.

Proposed responsibilities:

BleScanner: device discovery.
BleConnectionManager: connection, service discovery and notification subscriptions.
SspEventParser: packet decoding and validation.
MainActivity: connection status, incoming events and current SSP state.

Initial visible information:

BLE connection status.
Latest event sequence and timestamp.
Latest event.
Current state: NORMAL or ELEVATED.

A valid MOTION_EVENT triggers NORMAL → ELEVATED.

The application must not trust arbitrary received bytes. It should validate payload length, version, event type and field ranges before applying state changes.

Android permissions, BLE lifecycle behavior, disconnect handling and reconnection must be addressed for the actual Android SDK and target configuration.

10. Immediate engineering strategy

The development strategy changes from script-by-script conversational generation to batch-oriented implementation.

Principles:

Inspect the real repository and environment before generating code.
Preserve the existing known-good Hello World application and verification script.
Reuse an appropriate, available Zephyr BLE sample rather than implementing BLE from scratch.
Establish a single authoritative protocol specification before generating both endpoints.
Generate coherent groups of files with explicit interfaces and acceptance tests.
Compile firmware and Android code automatically after each batch.
Fix integration problems at the batch level rather than restarting the entire project.
Use synthetic events to validate BLE and Android independently of the physical IMU.
Integrate the real sensor behind the existing acquisition interface.
Commit each verified increment and preserve logs and test evidence.

The objective is to minimize implementation delay without sacrificing reproducibility or correctness.

11. Proposed route to the first end-to-end demonstration
A. Baseline audit

Inspect the repository, toolchain, Zephyr version, board identifier, existing scripts and Android project status.

B. BLE feasibility probe

Build and flash an appropriate existing Zephyr BLE peripheral sample. Confirm advertising, phone discovery, connection and characteristic access with a BLE diagnostic app.

C. SSP firmware batch

Generate the synthetic sensor pipeline, logical event model, GATT service, advertising and notification handling as a coherent firmware change.

D. Protocol validation

Document the UUIDs and packet encoding. Test representative valid and invalid payloads, sequence behavior and repeated events.

E. Android receiver batch

Generate the minimal Kotlin BLE receiver, parser, UI state model and tests against the same protocol definition.

F. Synthetic end-to-end test

Demonstrate:

Synthetic event
    -> nRF52840
    -> BLE notification
    -> Android validation
    -> NORMAL -> ELEVATED
    -> visible event/state

G. Physical sensor integration

Identify the actual IMU, configure its Zephyr driver and bus, replace the simulator, and calibrate motion detection experimentally.

H. Repeatability and checkpoint

Run the complete demonstration repeatedly, record logs and results, and commit a reproducible state.

A successful synthetic demonstration proves the communication and Edge path, but does not by itself complete physical sensor integration or M2.

12. Acceptance criteria
Device and firmware
The firmware builds reproducibly from a clean checkout.
The device advertises and exposes the expected GATT service.
Synthetic events are generated with valid sequence and timestamp fields.
Notifications are delivered when a client subscribes.
Sensor failures are distinguishable from valid measurements.
Physical movement is eventually demonstrated using the actual IMU.
Android Edge
The application discovers and connects to the SSP device.
It subscribes to the correct characteristic.
It rejects invalid or malformed events.
It displays valid events and updates state deterministically.
Disconnect and reconnect behavior is tested.
End-to-end
Physical movement produces a valid device event.
The event reaches Android over BLE.
Android validates the event and changes the state.
The resulting state and event are visible.
The sequence is repeatable and supported by saved test evidence.
13. Initial KPIs

Instrument the system to measure:

Movement detection latency.
Device-to-Edge latency.
Notification rate.
Packet loss.
False-positive and false-negative rates.
Sampling rate.
Payload size.
Device and Edge processing time.
Energy consumption when appropriate.

Do not claim KPI success until a measurement procedure, test conditions and results are recorded.

14. Explicitly deferred features

Do not implement these in the first BLE vertical slice unless new evidence makes one essential:

GNSS and geofencing.
Cellular communication.
Cloud backend and database.
Dashboard and alerting.
Machine learning and predictive risk models.
Multi-device orchestration.
Production security architecture.
Production power optimization.
Advanced adaptive monitoring.
15. Repository evidence and decision rules

Every future session should distinguish:

Verified: supported by a successful build, test, hardware observation or saved evidence.
Proposed: a design decision or implementation suggestion not yet tested.
Unknown: requires repository inspection or a hardware/environment check.
Blocked: cannot proceed without a specific missing dependency.

Do not infer implementation status from this document alone.

Before generating code, inspect the current repository and reconcile this bootstrap with its actual files, installed toolchain and existing test results.

16. Instructions for future LLM sessions

Use this file as project context, not as proof that proposed code has already been implemented.

The working priority is to achieve a synthetic-event BLE-to-Android demonstration quickly, then integrate and validate the physical sensor.

Prefer large, coherent, testable code batches; shared protocol definitions; automated builds; deterministic tests; minimal manual file editing; and Git checkpoints.

Do not reintroduce slow script-by-script conversational development unless a specific technical failure requires a narrowly scoped fix.

Next required action: perform a repository and environment audit, then prepare a batch-generation manifest with file ownership, interfaces, dependencies, tests and acceptance criteria.
