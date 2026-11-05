Excellent suggestion. Based on the SSP specification, the BLE validation work completed, and the current hardware inventory, I'd realign both the report and the roadmap around the actual SSP objective:

Demonstrate distributed Device→Edge intelligence, not merely BLE telemetry.

1. Revised Milestone 1 Report Draft
Title

SmartSecurePerimeter (SSP) - Device to Edge BLE Communication Validation

Student: Louis-Marie Loe

Course: Internet of Things Laboratory

1. Executive Summary

The objective of the first SSP milestone was to establish and validate communication between an IoT Device and an Edge node using Bluetooth Low Energy (BLE).

The SmartSecurePerimeter (SSP) architecture aims to detect physical events at the Device layer, transmit relevant information to an Edge application, and eventually support adaptive monitoring and intelligent decision-making.

For this milestone, the Nordic nRF52840-DK board acts as the Device while an Android smartphone acts as the Edge. The implementation demonstrates BLE advertising, discovery, connection establishment, telemetry transfer using GATT notifications, and validation of a custom SSP event protocol.

The communication path was successfully validated through repeated transmission of SSP events and verification of the transmitted payload structure.

2. SSP Architecture

The SSP architecture follows a layered model.

                    SSP Architecture

+-------------------------------------------------+
|                  EDGE LAYER                     |
|                                                 |
| Android Smartphone                              |
| BLE Client                                      |
| SSP Validation Logic                            |
+---------------------^---------------------------+
                      |
                      | BLE Notifications
                      |
+---------------------v---------------------------+
|                 DEVICE LAYER                    |
|                                                 |
| Nordic nRF52840-DK                              |
| Zephyr RTOS                                     |
| SSP Event Generator                             |
| BLE GATT Server                                 |
+-------------------------------------------------+


Future versions will extend the architecture with Cloud services, data storage, dashboards, and machine-learning capabilities.

3. BLE Communication Design
3.1 BLE Roles

The system uses BLE Peripheral/Central communication.

BLE Peripheral
Nordic nRF52840-DK


Responsibilities:

Advertising
GATT Service hosting
Event generation
Notification transmission
BLE Central
Android smartphone


Responsibilities:

Device discovery
Connection establishment
Service discovery
Notification subscription
SSP event reception
3.2 BLE Operations Used
Advertising

Purpose:

Make the SSP device discoverable.

Advertising name:

SSP-POC

Connection

Purpose:

Establish a communication session between Device and Edge.

Observed:

CONNECTED count=1

Service Discovery

Purpose:

Allow the Edge to discover SSP capabilities exposed through GATT.

Characteristics

Purpose:

Provide a communication endpoint for SSP events.

Notifications

Purpose:

Allow the Device to push SSP events to the Edge without polling.

Workflow:

SSP Event
     ↓
Serialize
     ↓
Update Characteristic
     ↓
BLE Notification
     ↓
Android Edge

4. SSP Event Model

The SSP project defines a logical event representation independently of the BLE encoding.

Event Fields
device_id
sequence
timestamp
event_type
status
motion_value


This logical model allows the communication protocol to evolve while maintaining a stable semantic representation.

5. SSP BLE Payload Specification
Payload Length
17 bytes

Payload Structure
Byte 0      Protocol Version
Byte 1      Event Type
Byte 2      Status

Byte 3-6    Device ID

Byte 7-10   Sequence Number

Byte 11-14  Timestamp

Byte 15-16  Motion Value

Endianness
Little Endian

Example Payload
01 01 00 01 50 53 53
01 00 00 00
B1 1C 02 00
08 07


Decoded:

Version      = 1
Event Type   = MOTION_EVENT

Status       = OK

Device ID    = 0x53535001

Sequence     = 1

Timestamp    = 138417 ms

Motion       = 1800 mg

6. Validation Tests
Test 1: Device Discovery

Expected:

SSP-POC visible


Result:

PASS

Test 2: Connection Establishment

Expected:

Android connects successfully


Result:

PASS

Test 3: Notification Subscription

Expected:

Edge enables notifications


Result:

PASS

Test 4: Payload Validation

Expected:

Payload structure matches specification


Result:

PASS

Test 5: Notification Transport

Expected:

NOTIFY rc=0


Result:

PASS

Test 6: Sequence Validation

Observed:

1
2
3
...
86


Result:

PASS

Test 7: Timing Validation

Observed:

5.000 second interval


Result:

PASS

7. Known Issue

During extended operation an intermittent Zephyr kernel fault was observed.

Observed after:

86 notifications


Fault address:

0x000107c4


The issue appears independent of payload serialization and notification delivery, since all notifications were successfully transmitted prior to the fault.

This issue remains under investigation.

8. Conclusion

The Device→Edge communication objectives of the SSP architecture have been successfully demonstrated.

The Nordic nRF52840-DK advertises over BLE, establishes connections with an Android Edge device, transmits SSP telemetry events through GATT notifications, and delivers correctly formatted payloads according to the SSP protocol specification.

The implemented solution provides the communication foundation required for the next SSP milestone: Android-side SSP event validation and state management.

2. Updated SSP Roadmap
Current State
M0 Environment
✅ COMPLETE

M1 SSP Specification
✅ COMPLETE

M2 BLE Protocol + Device
✅ COMPLETE

M3 Device→Edge BLE Validation
✅ COMPLETE

Next Physical Architecture

Current:

nRF52840
     │
   BLE
     │
Android


Target:

Movement
    ↓
IMU
    ↓
nRF52840
    ↓
BLE
    ↓
Android SSP App
    ↓
NORMAL / ELEVATED


Later:

Movement
    ↓
IMU
    ↓
nRF52840
    ↓
BLE
    ↓
Android SSP App
    ↓
Risk Engine
    ↓
Cloud
    ↓
Dashboard

Hardware Connection Plan
Already Available
Device
Nordic nRF52840-DK

Edge
Android Phone

Future Sensor
IMU / Accelerometer

Optional Future Sensor
GPS Module


GPS should be considered Phase 2+ functionality because geolocation is explicitly outside current SSP scope.

Next Milestones
M4 - Android SSP Edge Application

Deliver:

Scan
Connect
Subscribe
Receive SSP Event
Display Event


Success:

Android displays received SSP event

M5 - IMU Integration

Hardware:

IMU
 ↓ I²C
nRF52840


Deliver:

Physical Motion
↓
MOTION_EVENT


Success:

Moving board generates SSP event

M6 - Edge State Engine

Implement:

NORMAL
↓ MOTION_EVENT
ELEVATED


Success:

Android changes state automatically

M7 - Adaptive Monitoring

Implement:

NORMAL
 → 10s notifications

ELEVATED
 → 2s notifications


Success:

Edge modifies monitoring behavior

3. Script-by-Script Development Plan
Phase A - Android BLE Reception
New Android Files
SSPBleScanner.kt

Responsibilities:

Scan SSP devices
Filter SSP-POC

SSPConnectionManager.kt

Responsibilities:

Connect
Disconnect
Reconnect

SSPGattClient.kt

Responsibilities:

Service Discovery
Characteristic Discovery
Notification Enable

SSPPacketDecoder.kt

Responsibilities:

Byte Array
↓
SSP Event Object

SSPEvent.kt

Represents:

data class SSPEvent(...)


Logical SSP model.

MainActivity.kt

Display:

Device Status

Current SSP State

Last Event

Sequence

Timestamp

Phase B - IMU Integration
Create
device/firmware/src/imu.c
device/firmware/src/imu.h


Responsibilities:

Sensor Initialization
Sensor Reading
Motion Magnitude

Create
device/firmware/src/motion_detector.c
device/firmware/src/motion_detector.h


Responsibilities:

Threshold Detection

Motion
↓
MOTION_EVENT

Modify
ssp_event.c


Current:

Timer-generated events


Future:

IMU-generated events

Phase C - Android SSP State Machine
Create
SSPStateMachine.kt


Logic:

NORMAL
↓
Valid Motion Event
↓
ELEVATED

Create
RiskEvaluator.kt


Initial implementation:

MOTION_EVENT
↓
ELEVATED

Phase D - Adaptive Monitoring
Add BLE Write Characteristic

Firmware:

ssp_gatt_service.c


Add:

Configuration Characteristic


Android:

SSPConfigManager.kt


Allows:

NORMAL
↓
Notify every 10 s

ELEVATED
↓
Notify every 2 s


This sequencing now aligns very closely with the official SSP specification and gives you a clear path from the current BLE-validated platform to the first complete SSP vertical slice.
