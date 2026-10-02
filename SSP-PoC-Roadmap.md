# SSP PoC Project Roadmap
### Target architecture

```
                         SSP PoC
┌───────────────────────────────────────────────────────────┐
│ DEVICE                                                    │
│ nRF52840-DK + sensors                                    │
│ Zephyr firmware                                           │
│ Sensor acquisition → local processing → BLE GATT          │
└───────────────────────────┬───────────────────────────────┘
                            │ BLE
                            ▼
┌───────────────────────────────────────────────────────────┐
│ EDGE                                                      │
│ Android / Kotlin                                         │
│ BLE → validation → features → risk → ML → policy         │
└───────────────────────────┬───────────────────────────────┘
                            │ HTTPS / MQTT
                            ▼
┌───────────────────────────────────────────────────────────┐
│ CLOUD                                                     │
│ Backend → database → analytics → dashboard → alerts      │
└───────────────────────────────────────────────────────────┘
```

 The roadmap should build this **bottom-up and interface-by-interface**, while continuously keeping an end-to-end demo working.

---

 # Phase 0 — Development environment setup

 **Goal: every team member can build, flash, debug, and version-control the project before implementing application functionality.**

 ### 0.1 Establish the project repository

 Create a Git repository immediately.

 Suggested structure:

```
ssp-iot-poc/
│
├── device/
│   ├── firmware/
│   ├── drivers/
│   ├── protocol/
│   └── tests/
│
├── edge/
│   ├── android/
│   ├── protocol/
│   └── tests/
│
├── cloud/
│   ├── backend/
│   ├── database/
│   ├── frontend/
│   └── tests/
│
├── ml/
│   ├── datasets/
│   ├── notebooks/
│   ├── training/
│   └── models/
│
├── docs/
│   ├── architecture/
│   ├── protocol/
│   ├── testing/
│   └── reports/
│
└── README.md
```

 Establish:

 - `main`/`develop` strategy or an equivalent simple workflow.
- Feature branches.
- Pull requests/code review.
- Issue/task tracking.
- Basic `.gitignore`.
- README with setup instructions.

 ### 0.2 Device development environment

 Install and verify:

 - VS Code.
- PlatformIO and/or the Nordic/Zephyr development tooling selected by the team.
- Zephyr SDK/toolchain.
- Git.
- nRF52840-DK support.
- USB/debug drivers if required.
- Serial-terminal software.

 **Deliverable:** a minimal Zephyr application successfully compiled and flashed to the nRF52840-DK.

 ### 0.3 Verify the board

 Perform the classic first test:

```
PC
 ↓ USB
nRF52840-DK
 ↓
Firmware
 ↓
LED / serial output
```

 Verify:

 - Flashing works.
- Debugging works.
- Serial output works.
- Board can be recovered/reflashed.
- Each team member can reproduce the process.

 ### 0.4 Android environment

 Install:

 - Android Studio.
- Required Android SDK.
- Kotlin.
- Emulator/device configuration.
- A physical Android phone for BLE testing.

 Create a minimal application that:

 - Builds successfully.
- Runs on the phone.
- Requests the required BLE permissions.
- Can be debugged from Android Studio.

 ### 0.5 Cloud development environment

 Initially keep this simple.

 Set up:

 - Backend development environment.
- Database.
- API testing tool.
- Local/cloud deployment method.

 You don't need to commit to a sophisticated commercial cloud platform yet.

 ### Phase 0 exit criterion

 **Every member can independently:**

```
Clone repository
      ↓
Build firmware
      ↓
Flash nRF52840
      ↓
Run Android application
      ↓
Run backend
```

 Do not proceed until this works.

---

 # Phase 1 — Freeze the SSP PoC use case

 Before buying/connecting lots of sensors, define exactly what the prototype demonstrates.

 ### Define

 1. Target SSP scenario.
2. Protected zone/perimeter concept.
3. Device.
4. Events.
5. Risk levels.
6. Edge decisions.
7. Cloud actions.
8. User-visible outputs.

 For example:

```
NORMAL
   ↓
movement detected
   ↓
SUSPICIOUS
   ↓
persistent/abnormal event
   ↓
HIGH RISK
   ↓
Immediate cloud event
```

 ### Deliverables

 Produce:

 - One-page use-case description.
- System context diagram.
- Main sequence diagram.
- List of events.
- Initial risk model.
- Initial KPI list.

 ### Exit criterion

 Everyone can answer:

 > **What exactly is our PoC detecting, predicting, and demonstrating?**

---

 # Phase 2 — Hardware and sensor integration

 Now connect the actual sensors.

 I'd start with:

 **nRF52840-DK + IMU/accelerometer + button/tamper input**

 Additional environmental sensors can come later if justified by the use case.

 ### Implement

```
Sensor
 ↓
Zephyr driver
 ↓
Raw measurement
 ↓
Filtering
 ↓
Normalized data
```

 Test each sensor independently.

 ### Deliverables

 - Sensor wiring diagram.
- Sensor configuration.
- Firmware driver/configuration.
- Raw-data test.
- Sampling-rate measurement.

 ### Exit criterion

 The nRF52840 reliably produces timestamped sensor measurements.

---

 # Phase 3 — Define the SSP data model

 Do this **before building the BLE protocol**.

 Define a canonical internal representation.

 For example:

```
{
  "device_id": "SSP-001",
  "timestamp": 0,
  "sequence": 0,
  "motion": {
    "x": 0.0,
    "y": 0.0,
    "z": 0.0
  },
  "event": "NONE",
  "risk": 0
}
```

 Decide:

 - Device ID.
- Timestamp.
- Sequence number.
- Sensor values.
- Units.
- Event types.
- Risk representation.
- Error/status fields.

 This becomes the common language between Device, Edge and Cloud.

---

 # Phase 4 — Design the BLE protocol

 Now translate the data model into an efficient BLE representation.

 Define:

 ### GATT structure

 For example:

```
SSP BLE Service
│
├── Sensor Data Characteristic
├── Event Characteristic
├── Status Characteristic
└── Configuration Characteristic
```

 Define:

 - Service UUID.
- Characteristic UUIDs.
- Read/write/notify properties.
- Notification behavior.

 ### Define the packet

 For example:

```
Byte 0       Protocol version
Byte 1       Message type
Byte 2       Device ID
Byte 3       Sequence
Bytes 4–7    Timestamp
Bytes 8–...  Payload
```

 The actual layout should be chosen based on your measurements and requirements.

 ### Deliverables

 Create a formal:

 **SSP BLE Protocol Specification v1.0**

 Include:

 - UUIDs.
- Packet diagrams.
- Byte ordering.
- Data types.
- Units.
- Valid ranges.
- Error handling.
- Example packets.

 This document will directly support the IoT Lab report requirement.

---

 # Phase 5 — Device → BLE

 Now combine the first three phases.

```
Sensor
 ↓
nRF52840
 ↓
Processing
 ↓
BLE GATT
 ↓
Notifications
```

 The board should:

 1. Read sensors.
2. Process data.
3. Construct SSP packets.
4. Advertise.
5. Accept a connection.
6. Notify the Edge.

 ### Testing

 Use an existing BLE inspection application first to verify:

 - Device discovery.
- Service discovery.
- Characteristics.
- Notifications.
- Packet contents.
- Notification frequency.

 Only after this works should you debug the Android application.

 ### Exit criterion

 **The PCB independently provides reliable SSP data over BLE.**

---

 # Phase 6 — Android Edge: BLE foundation

 Implement the Android BLE client.

 Architecture:

```
BLE Scanner
    ↓
Device discovery
    ↓
GATT connection
    ↓
Service discovery
    ↓
Characteristic subscription
    ↓
Notification receiver
```

 Handle:

 - Permissions.
- Scanning.
- Connection/disconnection.
- GATT errors.
- Reconnection.
- Notifications.
- Packet decoding.

 ### Important

 Keep the BLE layer separate from the rest of the application.

 For example:

```
BLEManager
     ↓
PacketDecoder
     ↓
SSPDataModel
     ↓
RiskEngine
```

 This will make the system much easier to test.

---

 # Phase 7 — First complete vertical slice

 This is a **major milestone**.

 Before implementing ML or sophisticated adaptive behavior, make this work:

```
Sensor
 ↓
nRF52840
 ↓ BLE
Android
 ↓
Display measurement
```

 For example:

 > Move the device → Android receives motion data → Android displays it.

 Record a short demonstration video/test evidence.

 ### Why this milestone matters

 You now have a functioning **Device → Edge** SSP prototype.

 If later Cloud/ML work goes wrong, you still have a working foundation.

---

 # Phase 8 — Edge data processing

 Turn Android from a BLE receiver into an SSP Edge.

 Implement:

 ### Data validation

 - Packet version.
- Device ID.
- Sequence number.
- Timestamp.
- Range checking.
- Missing packets.

 ### Feature extraction

 For motion, for example:

```
Raw X/Y/Z
    ↓
Magnitude
    ↓
Windowing
    ↓
Mean
Variance
Peak
Movement duration
```

 ### Local event detection

 Start with deterministic rules before ML.

 For example:

```
IF motion > threshold
    → MOTION_EVENT

IF tamper button activated
    → TAMPER_EVENT
```

 This gives you a working baseline against which the ML system can later be compared.

---

 # Phase 9 — Risk model

 Define the SSP risk model.

 Keep the first version transparent.

 For example:

```
Risk =
    motion contribution
  + event contribution
  + context contribution
  + historical contribution
```

 Then classify:

```
0–30    LOW
31–70   MEDIUM
71–100  HIGH
```

 The exact thresholds should be justified experimentally rather than chosen arbitrarily.

 This gives you a bridge between:

 **Sensing → Interpretation → Assessment**

---

 # Phase 10 — Adaptive monitoring

 Now implement one of the most distinctive SSP concepts.

 Define modes such as:

```
NORMAL
ELEVATED
CRITICAL
```

 Each mode changes:

 - Sampling frequency.
- Processing frequency.
- BLE notification frequency.
- Cloud transmission frequency.

 Example:

```
LOW RISK
  ↓
Normal sampling
  ↓
Periodic Cloud update

MEDIUM RISK
  ↓
Higher sampling
  ↓
More frequent Edge processing

HIGH RISK
  ↓
Event-driven sensing
  ↓
Immediate Cloud transmission
```

 This is where the PoC begins demonstrating **adaptive resource management**, rather than simply IoT connectivity.

---

 # Phase 11 — Edge ML

 Only after the deterministic system works should you introduce ML.

 ### Build the dataset

 Collect representative:

 - Normal activity.
- Movement.
- Sudden movement.
- Tamper events.
- Other relevant states.

 Label the data.

 ### Train

 Start with a relatively simple model.

 Possible candidates include:

 - Decision tree.
- Random forest.
- Logistic regression.
- Small neural network.

 Don't optimize model complexity prematurely.

 ### Evaluate

 Measure:

 - Accuracy.
- Precision.
- Recall.
- F1.
- False positives.
- False negatives.
- Inference latency.

 ### Deploy

 Ideally:

```
BLE data
 ↓
Feature extraction
 ↓
ML inference
 ↓
Risk prediction
 ↓
Adaptive policy
```

 This directly implements SSP's **predictive edge processing** concept.

---

 # Phase 12 — Edge → Cloud

 Now connect the working Edge to the backend.

 Choose **one** protocol:

 - MQTT, or
- HTTP REST.

 Don't implement both unless there is a clear requirement.

 For example:

```
Android
   │
   │ HTTPS
   ▼
Backend API
```

 or:

```
Android
   │
   │ MQTT
   ▼
MQTT broker
   ↓
Backend
```

 Define the Edge-to-Cloud schema formally.

 Example:

```
{
  "device_id": "SSP-001",
  "timestamp": "...",
  "event": "MOTION_EVENT",
  "risk_level": "HIGH",
  "prediction": 0.87
}
```

---

 # Phase 13 — Cloud database

 Implement persistent storage.

 Your database should support:

 - Device identification.
- Timestamped measurements.
- Events.
- Risk levels.
- Predictions.
- System status.

 A conceptual schema could be:

```
DEVICES
   │
   ├── MEASUREMENTS
   │
   ├── EVENTS
   │
   └── PREDICTIONS
```

 Don't over-engineer the database.

 The important thing is that you can reliably answer:

 > What happened, when did it happen, which device produced it, and what did the system predict?

---

 # Phase 14 — Dashboard

 Build the user-facing interface.

 At minimum show:

 ### Current state

```
Device: SSP-001
Status: Connected
Risk: HIGH
Last update: 13:42:05
```

 ### Historical data

 Graphs for:

 - Sensor measurements.
- Risk level.
- Events.
- Predictions.

 ### Event view

```
13:41:58  Motion detected
13:42:01  Risk increased
13:42:05  HIGH-RISK prediction
13:42:05  Cloud event received
```

 This gives you the final:

 **Cloud → User**

 part of the architecture.

---

 # Phase 15 — Privacy and security PoC

 Don't leave this until the final week.

 Implement reasonable demonstrable controls.

 ### Device/BLE

 - Defined device identity.
- Validate packet structure.
- Reject malformed packets.

 ### Edge

 - Validate incoming data.
- Avoid transmitting unnecessary raw data.

 ### Cloud

 - Authentication where practical.
- HTTPS/TLS or secure MQTT configuration.
- Access control.
- Avoid exposing secrets in source code.

 ### Privacy

 Explicitly define:

 > What data is collected?

 > What stays on the device?

 > What stays on the Edge?

 > What reaches the Cloud?

 This is particularly relevant to SSP's **privacy-aware communication** principle.

---

 # Phase 16 — Energy and performance evaluation

 Now measure the system.

 ## Device KPIs

 - Sampling rate.
- BLE transmission rate.
- Estimated energy consumption.
- Battery-life estimate if applicable.

 ## Communication KPIs

 - Packet-loss rate.
- BLE connection stability.
- Payload size.
- Bytes transmitted.
- Edge-to-cloud latency.

 ## Intelligence KPIs

 - Event-detection accuracy.
- ML precision/recall/F1.
- False-positive rate.
- Inference latency.

 ## System KPIs

 - End-to-end latency.
- Availability.
- Cloud ingestion rate.
- Dashboard update latency.

 The important point is to produce **actual measurements**, not just theoretical claims.

---

 # Phase 17 — End-to-end validation

 Run complete scenarios.

 ### Scenario A — Normal

```
Device
 ↓
normal measurements
 ↓
BLE
 ↓
Edge
 ↓
low risk
 ↓
periodic Cloud update
 ↓
dashboard
```

 ### Scenario B — Suspicious activity

```
Motion
 ↓
BLE
 ↓
Edge feature extraction
 ↓
risk increases
 ↓
ML prediction
 ↓
adaptive monitoring
 ↓
Cloud update
```

 ### Scenario C — Critical event

```
Event
 ↓
Device
 ↓
BLE
 ↓
Edge
 ↓
HIGH RISK
 ↓
immediate transmission
 ↓
Cloud
 ↓
Dashboard alert
```

 Capture evidence for every scenario.

---

 # Phase 18 — Failure testing

 This is particularly important because SSP claims resilience.

 Test:

 - BLE disconnection.
- Edge restart.
- Cloud unavailable.
- Malformed BLE packet.
- Missing data.
- Duplicate packet.
- Sensor failure.
- Network interruption.

 Ask:

 > **What does SSP do when something goes wrong?**

 For example:

```
Cloud unavailable
       ↓
Edge continues monitoring
       ↓
Store selected events locally
       ↓
Cloud reconnects
       ↓
Synchronize
```

 Even a simplified version demonstrates the architectural principle.

---

 # Phase 19 — Final integration

 At this point freeze the architecture.

 Don't add major features.

 Verify:

```
Device
   ↓
BLE
   ↓
Edge
   ↓
ML / Risk
   ↓
Cloud
   ↓
Database
   ↓
Dashboard
```

 Perform a clean installation from the Git repository.

 This is important: **the final PoC should be reproducible by someone other than its original developer.**

---

 # Phase 20 — Documentation and reports

 The documentation should be produced continuously, not written at the end.

 ## Report 1

 Focus on:

 **Device + BLE + Edge**

 Include:

 - Architecture.
- Hardware.
- Firmware.
- Zephyr.
- Sensors.
- BLE GATT.
- BLE packet specification.
- Android BLE implementation.
- Testing.
- Evidence.

 ## Report 2

 Complete:

 - Backend.
- Database.
- Frontend.
- ML.
- Edge intelligence.
- Security/privacy.
- Testing.
- End-to-end results.
- Team contribution.
- Course feedback.

---

 # Suggested milestone structure

 I'd organize the entire project around these **10 major milestones**:

 | Milestone | Result |
| --- | --- |
| **M0** | Development environment working |
| **M1** | SSP PoC use case and requirements frozen |
| **M2** | Sensors working on nRF52840 |
| **M3** | BLE protocol + Device BLE working |
| **M4** | Android receives and displays device data |
| **M5** | Edge risk/event processing working |
| **M6** | Adaptive monitoring + ML working |
| **M7** | Cloud + database working |
| **M8** | Dashboard + alerts working |
| **M9** | End-to-end validation + KPIs |
| **M10** | Final reproducible PoC \+ documentation |

The **critical milestone is M4**. At that point you already have a functioning Device → BLE → Edge system. Everything after that is incremental.

---

 # Recommended team workflow

 I would **not** have three people work completely independently on Device, Edge and Cloud for the whole semester. That creates integration risk.

 Instead:

 ### Initial stage

 Everyone participates in:

 - Environment setup.
- Architecture.
- Protocol design.
- First Device → Edge integration.

 ### Then specialization

 **Person A — Device**

```
Zephyr
Sensors
BLE
Power
Device testing
```

 **Person B — Edge/AI**

```
Android
BLE client
Risk engine
ML
Adaptive monitoring
```

 **Person C — Cloud**

```
Backend
Database
Dashboard
Cloud testing
```

 Then periodically rotate into **integration sessions**, where all three work together on the complete pipeline.

---

 # A practical "definition of done"

 I'd make this your final acceptance criterion:

 > **A sensor event generated on the nRF52840-DK must be transmitted over BLE, received and interpreted by the Android Edge, processed into an SSP risk/event decision, transmitted to the Cloud, persisted in the database, and displayed on the dashboard, with measurable latency and test evidence.**

 And for the more advanced SSP behavior:

 > **A change in the detected context/risk must cause a measurable change in the monitoring or communication policy.**

 If you can demonstrate those two things reliably, your PoC will have a very clear connection between the **SSP design philosophy** and the **actual IoT Lab implementation**.

 ### Recommended development order

 The most important thing is **not to start with ML or the dashboard**. Start here:

 **Environment → Board → Sensor → BLE → Android → End-to-end Device/Edge → Risk logic → Adaptive behavior → ML → Cloud → Database → Dashboard → Security/privacy → KPI validation.**

 That ordering minimizes integration risk and ensures you have a working prototype at every major stage.
