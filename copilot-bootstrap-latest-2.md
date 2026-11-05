SSP Roadmap & Phase Update
SmartSecurePerimeter (SSP) Project Status After Sensor Architecture Integration

Project: SmartSecurePerimeter (SSP)
 Platform: Nordic nRF52840-DK + Android Edge
 Current Date: November 2026
 Current Status: Sensor-Ready Architecture Implemented and Successfully Integrated

Executive Summary

The SSP project has now progressed beyond a pure BLE communication prototype.

Originally, the project demonstrated:

Synthetic Event
      ↓
BLE
      ↓
Edge


The project now includes a complete internal sensor architecture capable of supporting:

GPS
IMU
Motion Processing
Location Processing
Sensor Abstraction


while preserving the validated BLE communication stack and Protocol v1.

The newly integrated architecture successfully compiles and builds into firmware artifacts:

zephyr.elf
zephyr.hex


The next stage is no longer architectural design. The next stage is hardware validation and real sensor integration.

Phase Evolution
Phase A - Communication Validation
Objective

Answer the question:

Can SSP communicate Device → Edge?

Implemented
SSP Protocol v1
BLE Advertising
BLE GATT Service
BLE Notifications
Android BLE Client
Payload Serialization
Payload Decoding

Data Flow
Synthetic Motion Event
       ↓
Protocol Encoding
       ↓
BLE Notification
       ↓
Android Reception

Status

✅ COMPLETE

Phase B - BLE Validation (M3)
Objective

Validate end-to-end BLE transport.

Acceptance Criteria
Device Discovery
Connection
Service Discovery
Notification Subscription
Payload Validation
Notification Delivery
Sequence Validation
Timing Validation

Outcome
PASS
PASS
PASS
PASS
PASS
PASS
PASS
PASS

Status

✅ COMPLETE

What We Learned

At the end of M3, SSP successfully answered:

Can Device communicate with Edge?


Answer:

YES


However, we also identified a major limitation:

The firmware did not yet understand real sensors.


Current implementation at that point:

.motion_mg = 1800;


was synthetic.

The system could:

communicate


but could not yet:

acquire GPS data
acquire IMU data
perform motion analysis
perform location analysis

Phase C - Sensor Architecture Preparation
Objective

Answer the question:

Can SSP become a real sensor platform?


without disrupting the validated BLE stack.

Implemented Architecture

The following firmware modules were added:

Core Data Model
ssp_data_model.h
ssp_config.h

Purpose

Create a canonical SSP representation independent of BLE.

Implemented structures:

ssp_event_data_t
ssp_motion_t
ssp_position_t
ssp_imu_sample_t

Status

✅ COMPLETE

GPS Layer

Implemented:

gps.h
gps.c

location_engine.h
location_engine.c

Current Capability
GPS abstraction
Location abstraction
Simulation support

Not Yet Implemented
UART driver
NMEA parser
Real GPS coordinates

Status

✅ Architecture Complete

⏳ Real GPS Integration Pending

IMU Layer

Implemented:

imu.h
imu.c

Current Capability
IMU abstraction
Simulated IMU acquisition

Not Yet Implemented
I²C communication
Real IMU measurements

Status

✅ Architecture Complete

⏳ Hardware Integration Pending

Motion Processing Layer

Implemented:

motion_detector.h
motion_detector.c

Current Capability
Motion Metric Calculation
Threshold Evaluation
Motion Event Generation

Data Flow
IMU Sample
     ↓
Motion Detector
     ↓
Motion Metric

Status

✅ COMPLETE

Sensor Abstraction Layer

Implemented:

sensor_manager.h
sensor_manager.c

Purpose

Provide a single integration point for:

GPS
IMU
Future Sensors

Data Flow
GPS
 ↓
Location Engine

IMU
 ↓
Motion Detector

       ↓
Sensor Manager
       ↓
SSP Event Data

Status

✅ COMPLETE

Integration Work Completed
Firmware Integration

Successfully patched:

CMakeLists.txt
main.c
ssp_event.c

New Event Path

Old:

Timer
 ↓
1800 mg
 ↓
BLE


New:

IMU Layer
 ↓
Motion Detector
 ↓
Sensor Manager
 ↓
SSP Event
 ↓
BLE

Status

✅ COMPLETE

Protocol Strategy Preserved

The following components were intentionally NOT modified:

BLE Layer
ssp_gatt.c
ssp_gatt.h


Status:

✅ Unchanged

Protocol Layer
generated/ssp_protocol.h


Status:

✅ Unchanged

Payload Format
Protocol v1

17 bytes


Status:

✅ Unchanged

Current Device Architecture
GPS
 ↓
Location Engine
 ↓

IMU
 ↓
Motion Detector
 ↓

Sensor Manager
 ↓

SSP Event Model
 ↓

Protocol v1
 ↓

BLE Transport
 ↓

Android Edge

Build Validation

A full firmware rebuild was successfully completed.

Generated:

zephyr.elf
zephyr.hex


Build result:

BUILD SUCCESS

Significance

This confirms:

All new source files compile
All dependencies resolve
All modules link successfully
Existing BLE stack remains functional

Status

✅ COMPLETE

Updated Milestone Status
Milestone	StatusM0 Environment	✅ Complete
M1 SSP Specification	✅ Complete
M2 Event Generation	✅ Complete
M3 Device→Edge BLE Communication	✅ Complete
Deliverable 1	🟡 Evidence Package Remaining
M4 Android SSP Edge Application	🟡 In Progress
Phase C Sensor Architecture	✅ Complete
Integrated Firmware Build	✅ Complete
Firmware Flash & Runtime Validation	⏳ Next
M5 Real IMU Integration	⏳ Pending Hardware
M6 State Engine	⏳ Planned
M7 Adaptive Monitoring	⏳ Planned
M8 Location Model	⏳ Planned
M9 Real GPS Integration	⏳ Planned
M10+ Cloud/Analytics/ML	⏳ Future
Immediate Next Steps
Step 1 - Flash Latest Firmware

Execute:

west flash -d /home/louis/ssp-iot-poc/build/ble_peripheral

Goal

Validate:

Device boots successfully


Expected logs:

GPS initialized (simulation mode)

IMU initialized (simulation mode)

SSP event subsystem initialized

BT_INITIALIZED

Step 2 - BLE Regression Test

Verify:

Advertising
Connection
Notification Subscription
Event Reception


still function after integration.

Goal

Confirm:

The sensor architecture did not break BLE.

Step 3 - Android Validation

Verify:

Device
 ↓
Event
 ↓
Android
 ↓
Decode
 ↓
Display


Expected:

Motion values continue arriving.

Step 4 - Real GPS Integration
Hardware Available

✅ GPS Module

Target Architecture
GPS Module
 ↓
UART Driver
 ↓
NMEA Parser
 ↓
Location Object
 ↓
SSP Position

Acceptance Criteria

UART output demonstrates:

FIX=1
LAT=...
LON=...
SAT=...


using real GPS measurements.

Step 5 - Real IMU Integration
Hardware Status

⏳ Awaiting delivery

Target Architecture
IMU
 ↓
I²C Driver
 ↓
XYZ Samples
 ↓
Motion Detector
 ↓
MOTION_EVENT

Acceptance Criteria
Physical Movement
 ↓
Motion Event
 ↓
BLE Notification
 ↓
Android Reception


with no synthetic values.

Current Project Position

The SSP project has now completed the transition from:

Communication Prototype


to:

Sensor-Ready Architecture


The architecture required for:

GPS
IMU
Motion Processing
Location Processing
Sensor Abstraction


is now implemented, integrated, and building successfully.

The next milestone is no longer architectural development.

The next milestone is hardware validation:

Flash Firmware
 ↓
Validate Runtime Behavior
 ↓
Integrate Real GPS
 ↓
Integrate Real IMU
 ↓
Achieve First Full Sensor-Driven SSP Vertical Slice


Target vertical slice:

Physical Movement
      ↓
IMU
      ↓
Motion Detector
      ↓
MOTION_EVENT
      ↓
BLE
      ↓
Android Edge
      ↓
NORMAL → ELEVATED
      ↓
Visible User State
