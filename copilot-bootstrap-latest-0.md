SmartSecurePerimeter (SSP) Project Bootstrap Document
Project Baseline, Milestones, Architecture and Current Status

Project: SmartSecurePerimeter (SSP)
 Course: Internet of Things Laboratory
 Platform: Nordic nRF52840-DK + Android Edge
 Status: Device↔Edge BLE Communication Successfully Implemented and Validated

1. Project Vision

SmartSecurePerimeter (SSP) is an IoT architecture intended to demonstrate:

Distributed intelligence
Edge computing
Adaptive monitoring
Privacy-aware sensing
Event-driven communication
Risk-based decision making

The objective is not merely to build an IoT sensor node, but to demonstrate how information can be progressively transformed from raw sensor measurements into meaningful security and monitoring decisions.

2. Final SSP Target Architecture
                         SSP PoC
┌───────────────────────────────────────────────┐
│ DEVICE                                        │
│ nRF52840-DK + Sensors                         │
│ Zephyr RTOS                                   │
│ Sensor Acquisition                            │
│ Motion Processing                             │
│ Event Generation                              │
│ BLE Communication                             │
└───────────────────┬───────────────────────────┘
                    │ BLE
                    ▼
┌───────────────────────────────────────────────┐
│ EDGE                                          │
│ Android / Kotlin                              │
│ BLE Reception                                 │
│ Protocol Validation                           │
│ State Engine                                  │
│ Risk Engine                                   │
│ Adaptive Monitoring                           │
└───────────────────┬───────────────────────────┘
                    │ HTTPS / MQTT
                    ▼
┌───────────────────────────────────────────────┐
│ CLOUD                                         │
│ Backend                                       │
│ Database                                      │
│ Dashboard                                     │
│ Analytics                                     │
└───────────────────────────────────────────────┘

3. SSP First Vertical Slice

The first complete SSP demonstration must satisfy:

Physical Movement
       ↓
Accelerometer / IMU
       ↓
nRF52840 Firmware
       ↓
Motion Detection
       ↓
MOTION_EVENT
       ↓
BLE Notification
       ↓
Android Edge
       ↓
Packet Validation
       ↓
NORMAL → ELEVATED
       ↓
State Visible To User


This remains the primary acceptance path for the project.

4. Current Hardware Inventory
Device Layer
Available
Nordic nRF52840-DK

Edge Layer
Available
Android Smartphone


Acts as:

BLE Central
SSP Edge Node
State Engine Host
Future Risk Engine Host

Sensor Layer
Available
GPS Module


Current status:

Deferred


Used only in future milestones.

Pending
IMU / Accelerometer


Current status:

Expected hardware


Critical for SSP motion detection milestone.

5. SSP Logical Event Model

The BLE protocol is not the primary model.

The logical SSP model is.

Current canonical SSP event:

device_id
sequence
timestamp
event_type
status
motion_value


Future extension:

device_id
sequence
timestamp
event_type
status

motion

position
    latitude
    longitude
    altitude


Important principle:

Data Model
      ↓
Protocol
      ↓
BLE Transport


Not:

Sensor
      ↓
Protocol directly

6. Current BLE Protocol
Version 1

Payload Size

17 bytes


Structure:

Bytes	Field0	Protocol Version
1	Event Type
2	Status
3-6	Device ID
7-10	Sequence
11-14	Timestamp
15-16	Motion

Current Event Types:

NONE
MOTION_EVENT


Future Event Types:

NONE
MOTION_EVENT
LOCATION_EVENT
GEOFENCE_EVENT
TAMPER_EVENT

7. Project Milestones
M0 – Development Environment
Objective

Establish development infrastructure.

Deliverables
Firmware
Zephyr environment
Toolchain
Flashing pipeline
Android
Android Studio
Kotlin environment
BLE permissions
Repository
Git repository
Project structure
Acceptance Criteria

Every team member can:

Clone
Build
Flash
Debug
Run Android application

Status

✅ COMPLETE

M1 – SSP Specification
Objective

Define SSP use case and architecture.

Deliverables
Documents
SSP concept
Device→Edge architecture
Event model
State model
Acceptance criteria
Acceptance Criteria

Able to answer:

What does SSP detect?
What decisions does SSP make?
What demonstrates success?

Status

✅ COMPLETE

M2 – SSP Event Generation
Objective

Generate SSP motion events.

Deliverables
Firmware
SSP event subsystem
Event publisher
Periodic events
Event Model
MOTION_EVENT

Acceptance Criteria

The device autonomously generates SSP events.

Status

✅ COMPLETE

Current implementation:

Synthetic motion events

M3 – Device↔Edge BLE Communication
Objective

Demonstrate BLE transport between Device and Edge.

Deliverables
Device
BLE advertising
GATT service
Notification characteristic
Payload serialization
Connection handling
Edge
BLE discovery
BLE connection
Notification subscription
Protocol
Device ID
Sequence
Timestamp
Motion field
Acceptance Criteria
Device Discovery

PASS

Connection

PASS

GATT Discovery

PASS

Notification Subscription

PASS

Payload Validation

PASS

Notification Delivery

PASS

Sequence Validation

PASS

Timing Validation

PASS

Status

✅ COMPLETE

Deliverable 1
Title

Device→Edge BLE Communication Validation

Status

~90–95% Complete

Completed

✅ BLE advertising

✅ BLE connection

✅ SSP service

✅ GATT notifications

✅ Protocol encoding

✅ Protocol decoding

✅ Android BLE client

✅ Event transmission

Remaining
Evidence package
Screenshots
Test report
Demonstration procedure
8. Current Milestone
M4 – Android SSP Edge Application
Current Project Focus

This is the active milestone.

The BLE transport layer is already operational.

The next objective is turning Android from:

BLE receiver


into an

SSP Edge node

M4 Objectives

Implement:

Scan
↓
Connect
↓
Subscribe
↓
Receive Packet
↓
Decode Packet
↓
Display Event

Proposed Android Components
SSPBleScanner

Responsibilities:

Device Discovery

SSPConnectionManager

Responsibilities:

Connect
Disconnect
Reconnect

SSPGattClient

Responsibilities:

Service Discovery
Characteristic Discovery
Notification Enablement

SSPPacketDecoder

Responsibilities:

BLE Payload
↓
SSP Event

SSPEvent

Represents:

Canonical SSP Logical Event

Android UI

Display:

Connection Status
SSP State
Last Event
Sequence
Timestamp
Motion Value

M4 Acceptance Criteria

The test passes when:

Device sends SSP event
       ↓
Android receives packet
       ↓
Android decodes event
       ↓
Event displayed to user


Required evidence:

Screenshots
Logs
Repeatable demonstration
9. Next Milestone
M5 – IMU Integration
Objective

Replace synthetic motion generation.

Current:

Timer
↓
Motion = 1800


Target:

Physical Movement
↓
IMU
↓
Motion Detection
↓
MOTION_EVENT

Planned Components
imu.h / imu.c

Responsibilities:

IMU initialization
Raw measurement acquisition

motion_detector.h / motion_detector.c

Responsibilities:

X/Y/Z processing

Motion Metric

Threshold Detection

sensor_manager.h / sensor_manager.c

Responsibilities:

Sensor abstraction

IMU

GPS

Future sensors

M5 Acceptance Criteria

The test passes when:

Physical movement
↓
IMU
↓
MOTION_EVENT
↓
BLE Notification
↓
Android receives event


with no synthetic event generation.

10. Future Milestones
M6 – SSP State Engine

Implement:

NORMAL
↓ valid MOTION_EVENT
ELEVATED


Acceptance:

Android changes state automatically

M7 – Adaptive Monitoring

Implement:

NORMAL
 → 10 s notifications

ELEVATED
 → 2 s notifications


Acceptance:

Edge modifies monitoring policy

M8 – Location Model

Add:

LOCATION_EVENT
GEOFENCE_EVENT


Introduce GPS information into:

SSP Logical Data Model


without changing the current BLE protocol.

M9 – GPS Integration
GPS
↓ UART
nRF52840
↓
Location Object
↓
LOCATION_EVENT

M10+

Future work:

Cloud Backend
Database
Dashboard
Machine Learning
Security
Performance KPIs
End-to-End Validation

11. What Has Already Been Successfully Implemented?
Firmware

✅ SSP Event System

✅ BLE Advertising

✅ BLE Connection Handling

✅ SSP GATT Service

✅ Notification Characteristic

✅ Payload Serialization

✅ Sequence Numbers

✅ Timestamps

✅ Event Publishing

Android

✅ BLE Discovery

✅ BLE Connection

✅ Service Discovery

✅ Notification Subscription

✅ Protocol Decoding Infrastructure

Protocol

✅ Protocol Versioning

✅ Device Identification

✅ Motion Payload

✅ Event Type Support

✅ Sequence Validation

✅ Timing Validation

12. Recommended Immediate Next Steps
Priority 1

Complete Deliverable 1 evidence package.

Produce:

Test report
Screenshots
UART logs
Demonstration script
Priority 2

Finish Android SSP Edge Application (M4).

Implement:

Receive
Decode
Display

Priority 3

Implement SSP State Engine (M6-ready).

NORMAL
↓
MOTION_EVENT
↓
ELEVATED

Priority 4

Prepare IMU integration layer.

Create:

ssp_data_model.h

imu.h
imu.c

motion_detector.h
motion_detector.c

sensor_manager.h
sensor_manager.c

Priority 5

Integrate physical IMU once hardware arrives.

Final Status Summary
Milestone	StatusM0 Environment	✅ Complete
M1 SSP Specification	✅ Complete
M2 SSP Event Generation	✅ Complete
M3 BLE Validation	✅ Complete
Deliverable 1	🟡 Evidence Remaining
M4 Android SSP Edge	🟡 Current Milestone
M5 IMU Integration	⏳ Pending Hardware
M6 State Engine	⏳ Planned
M7 Adaptive Monitoring	⏳ Planned
M8 GPS Awareness	⏳ Planned
M9 GPS Integration	⏳ Planned
M10+ Cloud/ML	⏳ Future
Project Position

The project has successfully validated the Device↔Edge BLE communication foundation. The immediate objective is to complete the Android SSP Edge application and prepare the architecture for IMU-based motion detection, enabling the first complete SSP vertical slice:

Movement
    ↓
IMU
    ↓
MOTION_EVENT
    ↓
BLE
    ↓
Android SSP Edge
    ↓
NORMAL → ELEVATED
    ↓
Visible User State
