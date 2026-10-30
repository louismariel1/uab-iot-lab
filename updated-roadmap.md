SSP PoC Project Roadmap (v2.0)

Updated after Milestone 1 BLE validation

This roadmap reflects lessons learned during Device↔Edge integration and prioritizes continuous end-to-end demonstrability over feature accumulation.

1. Project Vision

The SSP PoC demonstrates an intelligent IoT monitoring architecture where:

Device
  ↓
BLE
  ↓
Edge
  ↓
Cloud
  ↓
Dashboard


Sensor events generated on an nRF52840-DK are transmitted via BLE to an Android Edge application, processed into events/risk levels, forwarded to the Cloud, stored, and ultimately presented to the user.

2. Updated Architecture
                         SSP PoC

┌─────────────────────────────────────────────────────┐
│ DEVICE                                              │
│ Nordic nRF52840-DK                                 │
│ Zephyr RTOS                                         │
│ Sensors / Event Generator                           │
│ SSP Data Model                                      │
│ BLE GATT Server                                     │
└──────────────────────┬──────────────────────────────┘
                       │ BLE Notifications
                       ▼
┌─────────────────────────────────────────────────────┐
│ EDGE                                                │
│ Android App (Kotlin)                               │
│ BLE Client                                          │
│ Packet Decoder                                      │
│ Validation                                          │
│ Risk Engine                                         │
│ Adaptive Logic                                      │
└──────────────────────┬──────────────────────────────┘
                       │ HTTPS or MQTT
                       ▼
┌─────────────────────────────────────────────────────┐
│ CLOUD                                               │
│ Backend API                                         │
│ Database                                            │
│ Dashboard                                           │
│ Analytics / ML                                      │
└─────────────────────────────────────────────────────┘

3. Development Principles
Principle 1

Always maintain a working demo.

At any point in time:

Current feature
    +
Previous working features


must remain functional.

Principle 2

Validate interfaces before implementing complexity.

Order:

Sensor
 ↓
Protocol
 ↓
BLE
 ↓
Android
 ↓
Cloud
 ↓
Dashboard


Not:

ML
 ↓
Dashboard
 ↓
Hope integration works later

Principle 3

Protocol first, implementation second.

Every interface must be formally documented before extensive coding begins.

4. Current Status Assessment
M0 – Development Environment
Objective

Establish reproducible development environment.

Status

✅ COMPLETE

Implemented:

Git repository
Zephyr environment
Build process
Flashing process
UART monitoring
Android BLE test tool (nRF Connect)
M1 – Use Case Definition
Objective

Freeze SSP scenario.

Deliverables
One-page use-case description
Context diagram
Event definitions
Risk definitions
Status

⚠ IN PROGRESS

Needs formal documentation.

M2 – Device Event Generation
Objective

Generate valid SSP events.

Current Implementation
Motion Event Generator
↓
Periodic Event
↓
5-second interval

Status

✅ COMPLETE (PoC)

Actual sensors can replace simulated events later.

5. Device Layer Roadmap
M3 – Protocol Design & BLE Validation
Objective

Validate BLE communication path.

Achievements

✅ Advertising

SSP-POC visible


✅ Connection

CONNECTED


✅ Service Discovery

✅ Notification Subscription

✅ Payload Serialization

✅ Payload Validation

✅ BLE Notification Delivery

✅ Long-duration Notification Testing

Current Protocol
Byte 0      Protocol Version
Byte 1      Event Type
Byte 2      Status

Byte 3-6    Device ID

Byte 7-10   Sequence Number

Byte 11-14  Timestamp

Byte 15-16  Motion Value

Status

✅ COMPLETE

This milestone forms the basis of Deliverable 1.

M4 – Device Stability Investigation
Objective

Investigate intermittent Zephyr crash.

Observed:

Kernel oops
Faulting PC:
0x000107c4

Investigation Tasks
Resolve crash location using addr2line
Analyze notification lifetime
Check buffer/queue exhaustion
Check timer callback behavior
Check memory corruption possibilities
Priority

MEDIUM

Reason

Communication functionality is already validated.

This should not block further project progress.

6. Edge Layer Roadmap
M5 – Android BLE Client
Objective

Replace nRF Connect with project-specific Android application.

Architecture
BLE Scanner
      ↓
Connection Manager
      ↓
Service Discovery
      ↓
Notification Receiver
      ↓
Packet Decoder
      ↓
UI

Required Features
BLE permissions
Device discovery
Connection handling
Automatic reconnection
Notification subscription
Raw packet reception
Exit Criterion
Android App
receives
real SSP packets

M6 – SSP Protocol Decoder
Objective

Decode notifications into meaningful objects.

Example:

{
  "version": 1,
  "event_type": 1,
  "device_id": "0x53535001",
  "sequence": 42,
  "timestamp": 343417,
  "motion": 1800
}

Exit Criterion

Android displays decoded SSP events.

M7 – Vertical Slice Demonstration
Objective

Complete first real Device→Edge workflow.

Motion Event
      ↓
nRF52840
      ↓ BLE
Android
      ↓
Display Event

Demonstration

Move device or generate event →

Android updates in real time.

Status

NEXT MAJOR MILESTONE

This is now the most important milestone of the project.

7. Edge Intelligence Roadmap
M8 – Validation & Rule Engine
Objective

Validate received data.

Checks:

Protocol version
Device ID
Sequence continuity
Timestamp consistency
Range validation
Example
missing packet
↓
warning generated

M9 – Risk Model v1
Objective

Transform events into risk levels.

Initial implementation:

LOW
MEDIUM
HIGH


based on deterministic thresholds.

Example:

Motion > threshold
↓
Risk += 20

Goal

Transparency before ML.

M10 – Adaptive Monitoring
Objective

Demonstrate changing behavior according to risk.

Example:

LOW
 ↓
Notify every 10 s

MEDIUM
 ↓
Notify every 5 s

HIGH
 ↓
Notify every 1 s

Importance

This demonstrates intelligence rather than simple telemetry.

8. Cloud Roadmap
M11 – Edge-to-Cloud Communication
Objective

Forward validated events to Cloud.

Select ONE protocol:

HTTP REST


or

MQTT

Recommendation

HTTP REST initially.

Simpler for demonstration purposes.

M12 – Backend API
Required Endpoints
POST /events

GET /events

GET /status

Exit Criterion

Events arrive successfully from Android.

M13 – Database

Store:

Device
Timestamp
Event
Risk
Status

Goal

Support historical analysis.

9. Visualization Roadmap
M14 – Dashboard

Display:

Current Status
Device Connected
Risk Level
Last Event

Historical Data

Graphs:

Events
Motion
Risk Evolution
M15 – Alerts

Examples:

HIGH RISK
↓
Dashboard Alert


Possible future extensions:

Email
Push Notification
10. ML Roadmap
M16 – Data Collection

Collect:

Normal
Movement
Tamper
Critical


datasets.

M17 – Model Training

Recommended:

Decision Tree


or

Random Forest


Avoid overly complex models.

M18 – Edge Inference
BLE Packet
↓
Feature Extraction
↓
ML Prediction
↓
Risk

Goal

Demonstrate predictive capability.

11. Security & Reliability
M19 – Security Controls
Device
Protocol validation
Version checking
Malformed packet rejection
Edge
Input validation
Secure storage
Cloud
Authentication
HTTPS
Secret management
M20 – Failure Testing

Test:

BLE disconnect
Android restart
Cloud unavailable
Malformed packet
Duplicate packet
Missing packet

Success Criterion

System degrades gracefully.

12. KPI Evaluation

Measure:

Device
Notification frequency
Payload size
Stability
BLE
Packet loss
Connection duration
Reconnection performance
Edge
Decode latency
Processing latency
Cloud
Ingestion latency
Dashboard update time
13. Deliverables Alignment
Deliverable 1 (Current)

Focus:

Device + BLE + Edge


Contents:

Architecture
nRF52840-DK
Zephyr
BLE GATT
SSP Protocol
Payload Design
Notification Mechanism
Validation Results
Testing Evidence

Status:

✅ Essentially complete

Deliverable 2

Focus:

Edge Intelligence
+
Cloud
+
Database
+
Dashboard
+
ML


Contents:

End-to-end architecture
Backend
Frontend
Storage
Analytics
Security
KPI measurements
Final Definition of Done

The SSP PoC is complete when:

Sensor Event
      ↓
nRF52840
      ↓ BLE
Android Edge
      ↓
Risk Assessment
      ↓
Cloud
      ↓
Database
      ↓
Dashboard
      ↓
User Alert


and a change in risk level causes a measurable change in monitoring or communication behavior.

This remains the primary acceptance criterion for the full SSP IoT project.
