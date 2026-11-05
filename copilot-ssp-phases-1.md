Excellent. Before generating any code, we should first establish the implementation order. The goal is to introduce the new SSP sensor architecture with minimum disruption to the already-validated BLE path.

I would implement the files in the following order.

Phase 1 - SSP Data Model Foundation

These files define the common language used by sensors, event generation, BLE and Android.

1
device/firmware/include/ssp_data_model.h


Purpose:

Central SSP event definitions.

Event types.
Device identifiers.
Payload structures.
Future protocol extensibility.


Status:

NEW FILE

Phase 2 - Sensor Abstraction Layer

The board should interact with sensors through one common interface.

2
device/firmware/include/sensor_manager.h


Purpose:

Public sensor API.


Status:

NEW FILE

3
device/firmware/src/sensor_manager.c


Purpose:

Sensor initialization.
Sensor polling.
Dispatch to IMU/GPS drivers.


Status:

NEW FILE

Phase 3 - IMU Layer

Implement now as a stub. Replace later with actual hardware implementation.

4
device/firmware/include/imu.h


Purpose:

IMU interface.


Status:

NEW FILE

5
device/firmware/src/imu.c


Purpose:

IMU implementation.

Initially:
simulated values

Later:
real I2C driver


Status:

NEW FILE

Phase 4 - GPS Layer

Prepare the architecture now even if GPS integration occurs later.

6
device/firmware/include/gps.h


Purpose:

GPS interface.


Status:

NEW FILE

7
device/firmware/src/gps.c


Purpose:

GPS implementation.

Initially:
stub

Later:
UART/NMEA parsing


Status:

NEW FILE

Phase 5 - Motion Processing

This converts IMU samples into SSP MOTION_EVENTs.

8
device/firmware/include/motion_detector.h


Purpose:

Motion detection API.


Status:

NEW FILE

9
device/firmware/src/motion_detector.c


Purpose:

Motion magnitude calculation.

Threshold logic.

Motion event generation.


Status:

NEW FILE

Phase 6 - Location Processing

Required later for GPS/geofence functionality.

10
device/firmware/include/location_engine.h


Purpose:

Location processing API.


Status:

NEW FILE

11
device/firmware/src/location_engine.c


Purpose:

Geofence logic.

Location interpretation.

Future LOCATION_EVENT support.


Status:

NEW FILE

Phase 7 - SSP Event Engine Refactor

At this point we connect sensors to the existing event system.

12
device/firmware/include/ssp_event.h


Purpose:

Update interfaces.

Support multiple event types.


Status:

MODIFY EXISTING FILE

13
device/firmware/src/ssp_event.c


Purpose:

Replace synthetic event generation.

Consume motion detector output.

Consume future location engine output.


Status:

MODIFY EXISTING FILE

Phase 8 - Main System Initialization

Integrate the new sensor architecture.

14
device/firmware/src/main.c


Purpose:

Initialize:

sensor_manager
imu
gps
motion_detector
location_engine


Status:

MODIFY EXISTING FILE

Phase 9 - BLE Payload Future-Proofing

Only minimal changes.

BLE is already validated.

15
device/firmware/src/ssp_ble_service.c


Purpose:

Support multiple event types.

Maintain MOTION_EVENT compatibility.


Status:

MODIFY EXISTING FILE

16
device/firmware/include/ssp_ble_service.h


Purpose:

Update serialization interfaces.


Status:

MODIFY EXISTING FILE

Android Preparation (Parallel Track)

These can be developed independently of the IMU.

17
edge/android/app/src/main/java/com/ssp/model/SSPEvent.kt


Purpose:

Android SSP event model.


Status:

NEW FILE

18
edge/android/app/src/main/java/com/ssp/protocol/SSPPacketDecoder.kt


Purpose:

ByteArray -> SSPEvent


Status:

NEW FILE

19
edge/android/app/src/main/java/com/ssp/ble/SSPBleScanner.kt


Purpose:

BLE scanning.


Status:

NEW FILE

20
edge/android/app/src/main/java/com/ssp/ble/SSPConnectionManager.kt


Purpose:

BLE connection management.


Status:

NEW FILE

21
edge/android/app/src/main/java/com/ssp/ble/SSPGattClient.kt


Purpose:

GATT discovery.
Notification subscription.


Status:

NEW FILE

22
edge/android/app/src/main/java/com/ssp/state/SSPStateMachine.kt


Purpose:

NORMAL
ELEVATED


Status:

NEW FILE

23
edge/android/app/src/main/java/com/ssp/state/RiskEvaluator.kt


Purpose:

Rule-based risk evaluation.


Status:

NEW FILE

Recommended Implementation Sequence

Implement strictly in this order:

1  ssp_data_model.h

2  sensor_manager.h
3  sensor_manager.c

4  imu.h
5  imu.c

6  gps.h
7  gps.c

8  motion_detector.h
9  motion_detector.c

10 location_engine.h
11 location_engine.c

12 ssp_event.h
13 ssp_event.c

14 main.c

15 ssp_ble_service.c
16 ssp_ble_service.h

17-23 Android files


This sequence gives you the cleanest migration path from:

Synthetic Event Generator
        ↓
SSP Event
        ↓
BLE


to:

IMU / GPS
        ↓
Sensor Processing
        ↓
SSP Event
        ↓
BLE
        ↓
Android State Machine


while preserving the BLE functionality that has already been validated.
