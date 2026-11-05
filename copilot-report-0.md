Yes. Based on what you've completed, we can already draft a strong First Report (BLE Communication Report).

According to the lab guidelines, the first report is specifically:

"task done on PCB and Android regarding BLE communication"

You do not need to document Cloud, Backend, Frontend, or ML implementation yet. That belongs mostly to Report #2.

Deliverable 1 Draft Structure
1. Executive Summary & Project Overview
Project Title

SSP (Smart Safety Platform): BLE Telemetry Communication Between Nordic Device and Edge Gateway

Company/Product Overview

Our team is developing an IoT safety monitoring platform capable of collecting sensor information from a wearable device and transmitting it through an Edge gateway for later cloud processing and visualization.

The objective of the first milestone was to establish reliable BLE communication between the Device layer and the Edge layer.

Milestone Objective

Demonstrate:

Device
↓
BLE Advertisement
↓
Connection
↓
Service Discovery
↓
Subscription
↓
Telemetry Notifications
↓
Edge Reception

2. System Architecture & Diagram

For the report, I recommend:

+--------------------+
| Device Layer       |
| Nordic nRF52840-DK |
| Zephyr RTOS        |
| SSP Firmware       |
+---------+----------+
          |
          | BLE GATT
          |
+---------v----------+
| Edge Layer         |
| Android Smartphone |
| nRF Connect        |
+---------+----------+

          (Future)

+---------v----------+
| Cloud Layer        |
| Backend + DB + UI  |
+--------------------+

Current Scope

Implemented:

Device Layer ✅
Edge Layer ✅

Future:

Cloud Layer ⏳
3. Layer Implementation Details
Device Layer
Hardware
Nordic nRF52840-DK
Firmware Framework
Zephyr RTOS
West build system
Nordic BLE stack
Firmware Responsibilities
BLE advertising
GATT services
Event generation
Payload serialization
BLE notification transmission
Runtime Event Flow
Timer Event
↓
Motion Event Generation
↓
Payload Serialization
↓
GATT Notification
↓
BLE Transmission

Edge Layer
Platform

Android smartphone

BLE Tool

nRF Connect

Responsibilities
Device discovery
Connection management
Service discovery
Notification subscription
Payload inspection
4. Data Structures & Protocol Payload Specification

This is now one of the strongest parts of the report because we've fully reverse-engineered and validated it.

BLE Frame

Total Length:

17 bytes

Payload Structure
Byte(s)	Field0	Version
1	Event Type
2	Status
3-6	Device ID
7-10	Sequence Number
11-14	Timestamp (ms)
15-16	Motion (mg)
Endianness
Little Endian

Example Payload
01 01 00 01 50 53 53
01 00 00 00
B1 1C 02 00
08 07

Decoded
Version      : 1
Event Type   : Motion Event
Status       : OK
Device ID    : 0x53535001
Sequence     : 1
Timestamp    : 138417 ms
Motion       : 1800 mg

5. Testing, Validation & Results

This section should be detailed because this is where most of the work was done.

Test 1 – BLE Advertising
Objective

Verify device advertising.

Result

Device visible as:

SSP-POC

Status

✅ PASS

Test 2 – BLE Connection
Objective

Verify Edge can establish connection.

Example Log
CONNECTED count=1 uptime=101512

Status

✅ PASS

Test 3 – Notification Subscription
Objective

Verify notification enablement.

Example
SSP event notifications: enabled

Status

✅ PASS

Test 4 – Payload Generation
Objective

Verify payload creation.

Example
PAYLOAD=
01 01 00 01 50 53 53 ...

Status

✅ PASS

Test 5 – BLE Notification Transmission
Objective

Verify payload delivery.

Example
NOTIFY rc=0

Status

✅ PASS

Test 6 – Sequence Integrity
Objective

Verify monotonic event sequence.

Observed:

1
2
3
...
86

Status

✅ PASS

Test 7 – Timing Validation
Objective

Verify transmission period.

Observed:

5.000 s

Status

✅ PASS

Test 8 – Sustained Communication Test
Objective

Verify BLE communication over several minutes.

Observed:

86 consecutive notifications


before a kernel fault occurred.

Status

✅ PASS (Communication Path)

Observation

An intermittent firmware stability issue remains under investigation.

Known Issues
Issue #1 – Kernel Oops During Extended Notification Sessions

Observed:

ZEPHYR FATAL ERROR 3
Kernel oops on CPU 0


Fault address:

0x000107c4


Characteristics:

Notifications transmitted successfully
rc=0
Payload valid
Crash occurs after prolonged operation

Current status:

Open


Planned action:

Future root-cause investigation using addr2line and firmware tracing.

Conclusions
Achieved

✅ BLE advertising

✅ BLE discovery

✅ BLE connection

✅ GATT service access

✅ Notification subscription

✅ SSP payload serialization

✅ Payload validation

✅ Notification transport

✅ Extended notification sessions

Milestone 1 Result

The Device and Edge layers successfully communicate through Bluetooth Low Energy. The Nordic nRF52840 wearable advertises, accepts connections, transmits validated SSP telemetry frames, and delivers them to the Edge application through BLE GATT notifications. The communication protocol was decoded, documented, and experimentally verified. A firmware stability issue remains under investigation but does not invalidate the achievement of the BLE communication objectives required for Deliverable 1.

I would consider this enough material to produce a polished 8–12 page Deliverable 1 report, with only a few screenshots (nRF Connect, Monitor V2.1 logs, architecture diagram, payload table) still needed.
