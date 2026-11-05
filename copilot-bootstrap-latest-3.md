Starting from "flash the latest firmware", I would focus on the shortest path to achieving the first real SSP sensor-driven vertical slice while minimizing risk.

Priority 1 — Flash & Smoke Test the New Firmware
Goal

Verify that the newly integrated sensor architecture actually runs on the nRF52840.

Actions
west flash -d /home/louis/ssp-iot-poc/build/ble_peripheral


Open UART logs.

Expected startup:

GPS initialized (simulation mode)

IMU initialized (simulation mode)

SSP event subsystem initialized

BT_INITIALIZED

SSP advertising started

Acceptance Criteria

✅ Device boots

✅ No hard fault / crash

✅ Advertising starts

✅ Periodic heartbeat visible

✅ Motion logs visible

Priority 2 — BLE Regression Testing
Goal

Prove the sensor architecture did not break the validated BLE stack.

Actions

Using:

nRF Connect (iPhone)


or

Android SSP Client


perform:

Scan
Connect
Discover Service
Enable Notifications
Receive Events

Acceptance Criteria

✅ SSP-POC advertising visible

✅ Connect succeeds

✅ Notifications enabled

✅ Events still arrive

✅ No disconnect loops

Priority 3 — Validate New Sensor Pipeline

This is the most important validation.

Goal

Prove events now originate from:

Sensor Manager
 ↓
Motion Detector


rather than:

motion_mg = 1800

Actions

Inspect UART:

MOTION detected=...
motion=...


Inspect Android payload.

The motion value should now reflect:

1200 + 900 = 2100


(from the simulated IMU values)

instead of:

1800

Acceptance Criteria

✅ Event payload contains new motion value

✅ Android receives updated value

✅ Full event chain still works

Priority 4 — Complete Deliverable 1 Evidence Package

This is high value because M3 is technically complete but still lacks formal evidence.

Produce
Screenshots
BLE Scan
Connection
Notifications Enabled
Received Event

Logs
UART Logs
Android Logs

Report

Include:

Protocol v1

17-byte payload

Device→Edge flow

Validation results

Acceptance Criteria

✅ M3 fully documented

✅ Submission-ready

Priority 5 — Android SSP Edge UI Completion (M4)

Current Android already has:

BLE Discovery
BLE Connect
Service Discovery
Notifications
Protocol Decode


Focus now on presentation.

UI Should Display
Connection State

Last Event Type

Sequence

Timestamp

Motion Value

Monitoring State

Acceptance Criteria
Device
 ↓
Event
 ↓
Decode
 ↓
Visible on screen

Priority 6 — GPS Hardware Integration

You already have the GPS module.

This is now the highest-impact firmware task.

Goal

Replace:

SSP_SIM_GPS_LAT_E7
SSP_SIM_GPS_LON_E7


with real coordinates.

Phase A

Implement:

GPS
 ↓
UART
 ↓
Raw NMEA
 ↓
UART Debug Output


Example output:

$GPRMC
$GPGGA

Acceptance Criteria

✅ UART receives NMEA sentences

Phase B

Implement:

NMEA
 ↓
Parser
 ↓
Latitude
Longitude
Satellites
Fix Status


Example:

FIX=1
LAT=41.548123
LON=2.107456
SAT=8

Acceptance Criteria

✅ Stable real-time coordinates

Priority 7 — SSP Location Object Validation

After GPS parsing works:

GPS
 ↓
Location Engine
 ↓
ssp_position_t

Validate
gps_get_position()


returns:

latitude
longitude
altitude
satellites

Acceptance Criteria

✅ Clean SSP location object

✅ GPS vendor independent

Priority 8 — IMU Hardware Arrival Preparation

The architecture is already ready.

Before the IMU arrives:

Verify
I²C pins
Power pins
Address
Driver availability


for the actual sensor.

Possible candidates:

LSM6DS3
MPU6050
BMI160

Prepare

Implement placeholders:

imu_init()
imu_read()


so only driver logic changes later.

Priority 9 — First Full SSP Vertical Slice

This is the first major project demonstration milestone.

Target:

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
Android
      ↓
NORMAL → ELEVATED
      ↓
Visible State Change

Acceptance Criteria

✅ No simulated motion

✅ Real movement required

✅ Android state changes automatically

Priority 10 — Location Event Design (M8 Preparation)

Before touching Protocol v1:

Design:

LOCATION_EVENT

GEOFENCE_EVENT


inside the SSP logical model.

Do not modify BLE payload yet.

Focus on:

Data Model
 ↓
Protocol
 ↓
Transport


which is already a core SSP design principle.

Recommended Immediate Roadmap
This Week
1. Flash firmware
2. Boot validation
3. BLE regression testing
4. Validate motion pipeline
5. Complete Deliverable 1 evidence package
6. Finish Android event display

Next Sprint
7. GPS UART integration
8. GPS NMEA parsing
9. SSP location object validation

After IMU Arrival
10. Real IMU driver
11. Motion detector with real data
12. Sensor-driven MOTION_EVENT
13. First complete SSP vertical slice


If I were prioritizing strictly by impact/risk, the next three actions would be:

#1 Flash latest firmware
#2 Validate BLE still works
#3 Integrate the real GPS module


because GPS hardware is already available, while IMU integration is currently blocked by hardware delivery.
