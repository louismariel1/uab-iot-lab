we are manipulating position and location data. when should we model this data and introduce it to the current code or how should this sensor data be handled in the SSP project?

This is actually a very important architecture question, and the SSP specification already gives us the answer.

My recommendation is:

Do not introduce GPS/location data into the BLE protocol or firmware implementation yet. Introduce location first into the SSP logical data model, then later expose it through the protocol when the GPS integration milestone begins.

Why?

Your current SSP implementation is focused on:

Movement
↓
MOTION_EVENT
↓
Device
↓
BLE
↓
Edge
↓
NORMAL → ELEVATED


The specification explicitly lists:

GNSS positioning
Geofencing


under:

Explicitly deferred functionality


Therefore GPS is currently a future SSP capability, not part of the current acceptance criteria.

When should location enter SSP?

I would introduce it in three stages.

Stage 1 - Data Model (now)

This is where it belongs first.

Today your logical event is roughly:

{
  "device_id": "SSP-001",
  "sequence": 123,
  "timestamp": 138417,
  "event_type": "MOTION_EVENT",
  "motion": 1800,
  "status": 0
}


I would future-proof it now:

{
  "device_id": "SSP-001",
  "sequence": 123,
  "timestamp": 138417,
  "event_type": "MOTION_EVENT",

  "motion": 1800,

  "position": {
      "latitude": null,
      "longitude": null,
      "altitude": null
  },

  "status": 0
}


Notice:

Position exists conceptually
but is empty.


This avoids redesigning the data model later.

Stage 2 - GPS Integration Milestone

Later when the GPS arrives:

GPS Module
↓
UART
↓
nRF52840
↓
NMEA Parser
↓
Location Object


At that point:

"position": {
   "latitude": 41.548,
   "longitude": 2.107,
   "altitude": 150
}


becomes populated.

This should happen after:

M5 IMU Integration
M6 State Engine


and before Cloud work.

Stage 3 - Edge Intelligence

This is where location becomes useful.

The Edge should not simply receive coordinates.

The Edge should interpret them.

Example:

GPS Position
↓
Inside authorized zone
↓
NORMAL


or

GPS Position
↓
Leaves perimeter
↓
LOCATION_EVENT
↓
ELEVATED


This is much more aligned with SSP.

How should GPS be represented?

I would not transmit raw NMEA strings.

Avoid:

$GPGGA,...
$GPRMC,...


inside SSP payloads.

Instead create a logical location structure:

{
  "latitude": 41.548123,
  "longitude": 2.107456,
  "fix": true,
  "accuracy": 3.2,
  "satellites": 8
}


Then serialize only what SSP needs.

Future SSP Event Types

Currently:

NONE
MOTION_EVENT


I would eventually extend to:

NONE
MOTION_EVENT
LOCATION_EVENT
GEOFENCE_EVENT
TAMPER_EVENT


without changing the architecture.

BLE Impact

This is important.

Your current payload is:

17 bytes


which is elegant and efficient.

Adding GPS directly would explode the payload size.

For example:

Latitude   4 bytes
Longitude  4 bytes
Altitude   2 bytes
Accuracy   2 bytes


could add:

12+ bytes


immediately.

Therefore I would not modify the current BLE packet until GPS functionality is actually implemented.

Maintain:

Protocol v1
17-byte MOTION_EVENT


Then later create:

Protocol v2


or

LOCATION_EVENT payload


for GPS-related messages.

Recommended SSP Evolution
Current
Motion
↓
MOTION_EVENT
↓
BLE
↓
Edge
↓
NORMAL/ELEVATED

Next
IMU
↓
MOTION_EVENT
↓
BLE
↓
Edge
↓
NORMAL/ELEVATED

Later
GPS
↓
LOCATION_EVENT
↓
BLE
↓
Edge
↓
Geofence Check
↓
NORMAL/ELEVATED

Advanced SSP
IMU + GPS
↓
Edge Correlation
↓
Risk Assessment
↓
Adaptive Monitoring
↓
Cloud

My recommendation

Right now:

✅ Add position fields to the SSP logical data model (as optional fields).

✅ Keep the current BLE protocol unchanged.

✅ Continue with IMU integration first.

✅ Integrate GPS only after the first SSP vertical slice (Motion → BLE → Android → State Transition) works.

That approach keeps the architecture clean, avoids protocol churn, and remains fully aligned with the SSP specification.
