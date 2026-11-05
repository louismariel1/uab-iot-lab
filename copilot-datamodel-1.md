This is actually one of the most important architectural questions in SSP right now.

My answer is:

The BLE communication layer is already being validated, even though the physical meaning of the data is still partially abstract.

However, for SSP to remain coherent, the semantic meaning of the data should now be explicitly specified.

What are we currently testing?

Today the system is effectively transmitting:

SSP Event


not

Raw Sensor Data


The payload contains:

version
event_type
status
device_id
sequence
timestamp
motion


Therefore the Device→Edge interface is already exchanging a structured SSP event.

In other words:

Current BLE Payload
    ≠
Raw accelerometer sample

Current BLE Payload
    =
Logical SSP Event

What is our current assumption?

The current assumption is approximately:

Movement has already been detected
      ↓
Device generates MOTION_EVENT
      ↓
Device sends event to Edge


This means the Device is not sending:

X = 0.152g
Y = 0.031g
Z = 0.984g


Instead it sends:

A motion event occurred.


plus supporting metadata.

Is the current payload abstract?

Partially.

The payload is:

Semantically defined


but

Sensor-source independent


For example:

Motion = 1800


currently means:

Derived motion value


but not necessarily:

Accelerometer X/Y/Z


because the IMU is not yet connected.

So today the payload behaves as:

Simulated MOTION_EVENT


which is perfectly acceptable for the BLE validation milestone.

Must the data type be specified now?
For BLE validation?

No.

The BLE communication milestone only needs:

Device
↓
Serialize
↓
Transmit
↓
Receive


Whether the payload represents:

temperature
motion
location
heart rate


does not change BLE itself.

For SSP architecture?

Yes.

From this point forward, I would strongly recommend formalizing the meaning.

SSP Logical Data Model

I would define SSP as transmitting:

Derived Events


rather than:

Raw Sensor Samples


because that aligns with the specification's privacy-aware philosophy.

The Device should answer:

"What happened?"

instead of:

"Here are all my sensor readings."

Recommended SSP Event Types
Current
NONE

MOTION_EVENT

Near Future
NONE

MOTION_EVENT

LOCATION_EVENT

TAMPER_EVENT

Later
NONE

MOTION_EVENT

LOCATION_EVENT

GEOFENCE_EVENT

TAMPER_EVENT

HIGH_RISK_EVENT

How should movement be modeled?

When the IMU arrives:

Instead of transmitting:

Accel X
Accel Y
Accel Z


continuously over BLE,

I would prefer:

Device
↓
Computes motion metric
↓
Determines movement detected
↓
Generates MOTION_EVENT
↓
Transmits concise event


This fits directly with the SSP document:

Raw Measurement
↓
Processing
↓
Event
↓
BLE
↓
Edge Decision

How should GPS be modeled?

The same principle applies.

Avoid:

Raw NMEA strings


over BLE.

Instead:

GPS Fix
↓
Location Model
↓
LOCATION_EVENT


Example:

{
  "event_type": "LOCATION_EVENT",
  "latitude": 41.548,
  "longitude": 2.107,
  "timestamp": 123456
}


or:

{
  "event_type": "GEOFENCE_EVENT",
  "zone": "PERIMETER_A",
  "status": "EXITED"
}


which is much more aligned with SSP.

What I would officially declare today

The SSP Device→Edge interface exchanges:

SSP Events


Each SSP event contains:

Device identity
Sequence information
Timestamp
Event type
Event-specific payload
Status information


Current implemented event:

MOTION_EVENT


Current motion data:

Synthetic / simulated


Future motion data:

IMU-derived


Future location data:

GPS-derived

Recommended project statement

I would put this in the architecture documentation:

The SSP Device does not primarily transmit raw sensor measurements. Instead, it transmits compact SSP events representing locally interpreted device observations. The current implementation supports the MOTION_EVENT type and uses a synthetic motion source for protocol validation. Future milestones will replace the synthetic source with IMU-derived measurements and extend the event model to support location-aware events generated from GPS data.

That statement is fully consistent with the SSP specification, your current implementation, and your planned roadmap.
