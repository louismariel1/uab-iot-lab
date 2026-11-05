Yes. Based on the architecture you've built and the BLE work you've completed, I would avoid touching the BLE layer as much as possible.

The goal should be:

Current

Synthetic Event Generator
         ↓
SSP Event
         ↓
BLE
         ↓
Android

Future

IMU / GPS
      ↓
Processing
      ↓
SSP Event
      ↓
BLE
      ↓
Android


In other words:

Replace the source of the event, not the communication stack.

Current SSP Architecture (Approximation)

Based on the files you've discussed previously, your firmware is roughly organized as:

main.c

ssp_event.c
ssp_event.h

ssp_ble_service.c
ssp_ble_service.h

notification handling

payload serialization

monitor tools


The next step is to insert a dedicated sensor-processing layer between sensors and SSP events.

Phase 1 - Create Sensor Abstraction Layer
New files
sensor_manager.h

Purpose:

Single point of entry to all sensors.


Responsibilities:

sensor_manager_init()

sensor_manager_poll()


Future:

IMU
GPS
Tamper Switch
Light Sensor
...

sensor_manager.c

Responsibilities:

Initialize all sensors.

Call each sensor driver.

Provide unified sensor data.


Architecture:

sensor_manager
    ↓
imu
    ↓
gps


This becomes:

main
 ↓
sensor_manager


instead of:

main
 ↓
individual sensors

Phase 2 - Create IMU Driver Layer

Create these now even before the IMU arrives.

imu.h

Defines:

struct imu_sample
{
    int16_t x;
    int16_t y;
    int16_t z;
};


Functions:

int imu_init(void);

int imu_read(struct imu_sample *sample);

imu.c

Current implementation:

Stub / simulated values


Future:

Real I2C IMU Driver


Benefit:

BLE work continues before hardware arrives.

Phase 3 - Create GPS Driver Layer
gps.h

Define:

struct gps_fix
{
    double latitude;
    double longitude;
    double altitude;

    bool valid;
};


Functions:

int gps_init(void);

int gps_get_fix(struct gps_fix *fix);

gps.c

Initial state:

Empty implementation


Future:

UART Driver
NMEA Parsing


This allows location support without redesign later.

Phase 4 - Create Motion Processing Layer

This is probably the most important missing piece.

motion_detector.h

Define:

bool motion_detected(void);


or

int motion_metric(struct imu_sample *sample);

motion_detector.c

Responsibilities:

Raw X,Y,Z
 ↓
Magnitude
 ↓
Filtering
 ↓
Threshold Check
 ↓
Motion Event


Current:

Uses simulated values


Future:

Uses real IMU

Phase 5 - Create Location Processing Layer
location_engine.h

Purpose:

Convert GPS data into SSP information.


Examples:

bool location_valid();
bool inside_perimeter();

location_engine.c

Future responsibilities:

GPS Fix
 ↓
Perimeter Logic
 ↓
LOCATION_EVENT

or

GEOFENCE_EVENT

Phase 6 - Refactor SSP Event Generation

This is probably where most future sensor integration happens.

Modify
ssp_event.c

Current:

Timer
 ↓
Synthetic Motion
 ↓
MOTION_EVENT


Target:

Motion Detector
 ↓
MOTION_EVENT

Location Engine
 ↓
LOCATION_EVENT

Introduce Event Sources

I would add something like:

enum ssp_event_type
{
    SSP_EVENT_NONE = 0,

    SSP_EVENT_MOTION = 1,

    SSP_EVENT_LOCATION = 2,

    SSP_EVENT_GEOFENCE = 3,

    SSP_EVENT_TAMPER = 4
};


This future-proofs the protocol.

Phase 7 - Refactor Payload Serialization
Modify
ssp_ble_service.c

or wherever payload packing currently resides.

Current:

Always serialize motion payload.


Target:

switch(event_type)
{
    MOTION_EVENT
    LOCATION_EVENT
    GEOFENCE_EVENT
}


Initially:

Only MOTION_EVENT implemented


but structure prepared.

Phase 8 - Add SSP Data Model

Create:

ssp_data_model.h

This becomes the official SSP language.

Example:

typedef struct
{
    uint32_t device_id;

    uint32_t sequence;

    uint32_t timestamp;

    uint8_t event_type;

    uint8_t status;

} ssp_event_header_t;


For motion:

typedef struct
{
    uint16_t motion;
} ssp_motion_payload_t;


For GPS:

typedef struct
{
    int32_t latitude;

    int32_t longitude;

} ssp_location_payload_t;

Phase 9 - Android Preparation

Before the sensors even arrive.

Create:

SSPEvent.kt
data class SSPEvent(...)

SSPPacketDecoder.kt

Responsibilities:

ByteArray
 ↓
SSPEvent


The Android app should decode:

MOTION_EVENT


first.

Later:

LOCATION_EVENT


without major changes.

Scripts I Would NOT Touch Yet

Leave alone until absolutely necessary:

ssp_ble_service.c


except minimal protocol extension.

BLE advertising code


Already validated.

Connection handling


Already validated.

Notification handling


Already validated.

My Recommended File Plan
Create
sensor_manager.c
sensor_manager.h

imu.c
imu.h

gps.c
gps.h

motion_detector.c
motion_detector.h

location_engine.c
location_engine.h

ssp_data_model.h

Modify
ssp_event.c
ssp_event.h


Main refactor location.

main.c


Initialize sensor manager.

ssp_ble_service.c


Only minor serialization improvements.

Do Not Touch (for now)
Advertising

Connection

Notification Transport

UART Monitor


because these are already validated and should remain stable while you integrate the IMU and GPS. This minimizes integration risk and keeps the BLE foundation you've worked hard to validate intact.
