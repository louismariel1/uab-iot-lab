Yes, and that requirement slightly changes the report strategy.

The lab is not asking for a generic project report yet. It is asking each student to explain:

How BLE communication was implemented
Which BLE mechanisms are used
Why they are used
How data is exchanged
What the payload structure looks like
Evidence that communication works

Because the delivery is individual, you should write it in your own words, even though the implementation was performed as a team.

What the evaluator will want to see

They are unlikely to care about:

Cloud architecture
Future ML plans
Business vision
Long-term roadmap

for this first deliverable.

They will care about:

Device discovers
↓
Advertises
↓
Edge connects
↓
GATT services discovered
↓
Notifications enabled
↓
Data transmitted
↓
Payload documented

Recommended Deliverable 1 Structure
1. Introduction
Objective

Explain that the purpose of the first milestone was to establish BLE communication between:

Nordic nRF52840-DK
(Device)

and

Android Edge Application
(nRF Connect)


Example wording:

The objective of this first milestone was to implement and validate Bluetooth Low Energy (BLE) communication between the Nordic nRF52840-DK board and an Edge device. The Edge device used during development was an Android smartphone running the nRF Connect application. The goal was to demonstrate device discovery, connection establishment, notification subscription, and telemetry transmission.

2. BLE Communication Architecture

Include a simple diagram:

+-------------------+
| Nordic nRF52840   |
| Zephyr Firmware   |
+---------+---------+
          |
          | BLE
          |
+---------v---------+
| Android Phone     |
| nRF Connect App   |
+-------------------+


Explain:

Device acts as BLE Peripheral.
Smartphone acts as BLE Central.
3. BLE Features Used

This section directly answers:

What parts of BLE operation do you use and what do you use them for?

Advertising

Used so the Edge can discover the device.

Observed:

SSP-POC


Purpose:

Device discovery.

Connection

Used so the smartphone can communicate with the board.

Observed:

CONNECTED count=1


Purpose:

Establish communication channel.

GATT Services

Used to expose application data.

Purpose:

Provide access to SSP telemetry.

Characteristics

Used to transport telemetry between devices.

Purpose:

Transmit structured sensor information.

Notifications

Used instead of continuous polling.

Purpose:

Device pushes data automatically.


Observed:

SSP event notifications: enabled

4. Data Exchange Mechanism

Explain the flow:

Device generates event
↓
Payload serialized
↓
Characteristic updated
↓
BLE Notification
↓
Edge receives packet


Observed:

NOTIFY rc=0


which indicates successful transmission.

5. BLE Payload Specification

This is arguably the strongest section because you have experimentally verified everything.

Total Length
17 bytes

Structure
Byte 0      Version
Byte 1      Event Type
Byte 2      Status

Byte 3-6    Device ID

Byte 7-10   Sequence Number

Byte 11-14  Timestamp (ms)

Byte 15-16  Motion Value (mg)

Endianness
Little Endian

Example Payload
01 01 00 01 50 53 53
01 00 00 00
B1 1C 02 00
08 07


Decoded:

Version      = 1
Event Type   = Motion Event
Status       = OK
Device ID    = 0x53535001
Sequence     = 1
Timestamp    = 138417 ms
Motion       = 1800 mg

6. Testing and Validation

This section is critical.

Test 1 – Device Discovery

Result:

Device visible as SSP-POC.


Status:

PASS

Test 2 – Connection

Result:

Connection established successfully.


Status:

PASS

Test 3 – Notification Subscription

Result:

Notifications successfully enabled.


Status:

PASS

Test 4 – Payload Validation

Result:

Payload structure decoded and verified.


Status:

PASS

Test 5 – Long Duration Transmission

Result:

86 consecutive notifications transmitted successfully.


Observed:

5 second interval


Status:

PASS

7. Issues Encountered

Be honest.

Example:

During extended notification testing, a Zephyr kernel fault was observed after approximately 86 notifications. Despite this issue, all transmitted packets before the failure were correctly formatted and delivered successfully. The problem appears to be related to firmware stability rather than BLE payload transmission and remains under investigation.

This actually strengthens the report because it demonstrates engineering rigor.

8. Conclusion

Something like:

The first milestone objectives were successfully achieved. The Nordic nRF52840-DK device advertises through BLE, accepts connections from an Android Edge device, supports notification-based communication, and transmits validated telemetry payloads. The payload structure was documented and experimentally verified. BLE communication between the Device and Edge layers is therefore considered operational and validated.

Can we already write Deliverable 1?

Yes.

Based on the evidence you now have:

Advertising ✅
Discovery ✅
Connection ✅
GATT ✅
Notifications ✅
Payload definition ✅
Payload verification ✅
Extended transmission test ✅

you have enough material to complete the first report, even though the long-term stability issue is not yet fully understood. The BLE communication objectives required by the assignment have been demonstrated convincingly.
