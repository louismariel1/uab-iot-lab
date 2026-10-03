# SmartSecurePerimeter (SSP) — Lab PoC Specification
## 1. PoC objective

Demonstrate a functional Device → Edge SSP architecture in which a physical movement event is sensed by an nRF52840-based device, locally interpreted, transmitted over Bluetooth Low Energy (BLE), validated by an Android Edge application, and converted into an SSP monitoring/risk state.

The PoC is intended to demonstrate the architectural principles of distributed intelligence, adaptive monitoring, privacy-aware communication, and measurable engineering rather than reproduce the complete production SSP system.

## 2. PoC scenario

A monitored device is operating in a defined monitoring context.

Under normal conditions, the device performs regular motion sensing.

When physical movement is detected:

NORMAL
   ↓
movement detected
   ↓
MOTION_EVENT
   ↓
Android Edge receives and validates event
   ↓
SSP state becomes ELEVATED


The initial PoC deliberately uses a simple deterministic rule. Machine learning, positioning/geofencing, cloud intelligence, and advanced risk modelling are subsequent stages.

## 3. System scope
Device

Hardware:

nRF52840-DK

Accelerometer/IMU

Optional button/tamper input in a subsequent increment

Firmware:

Zephyr RTOS

Sensor acquisition

Basic local motion processing

Event generation

BLE advertising and GATT communication

Edge

Platform:

Android / Kotlin

Functions:

BLE scanning and connection

GATT service discovery

Notification reception

Packet validation

Sequence/timestamp validation

Basic feature/event interpretation

SSP state/risk transition

Local display of current state and received events

Cloud

Out of scope for the first vertical slice.

Cloud integration will be introduced after the Device → Edge path is stable.

## 4. Device processing

The device shall periodically acquire accelerometer measurements:

Accelerometer
     ↓
Raw X/Y/Z
     ↓
Basic filtering / validation
     ↓
Motion metric
     ↓
Threshold-based event detection


The initial motion detector shall be deterministic and configurable.

The exact threshold and sampling parameters will be established experimentally during sensor integration rather than hard-coded into this specification.

## 5. SSP data model

The initial logical event shall contain at least:

device_id
sequence
timestamp
event_type
motion information / derived value
status


Initial event types:

NONE
MOTION_EVENT


The logical model is independent of the eventual BLE byte-level encoding.

## 6. BLE interface

The device shall expose an SSP-specific BLE GATT service.

The initial interface shall provide:

Device identification/status

Sensor/event data

Notification capability

The detailed UUIDs, characteristic properties, byte layout, data types, units, and versioning will be defined in the SSP BLE Protocol Specification v1.0 after the device data model has been validated.

## 7. Edge state model

The first Edge implementation shall use two states:

NORMAL
ELEVATED


Initial transition:

NORMAL
  │
  │ valid MOTION_EVENT
  ▼
ELEVATED


The state transition is intentionally deterministic and explainable.

More sophisticated risk levels and predictive models will be introduced later.

## 8. Adaptive monitoring — initial demonstration

The first PoC shall establish the mechanism for adaptive behavior but does not require a sophisticated policy.

At minimum, the architecture shall allow the Edge to request or configure a different monitoring policy based on state:

NORMAL
  ↓
normal sensing / notification policy

ELEVATED
  ↓
increased monitoring policy


The actual sampling and communication changes will be implemented and measured in a subsequent milestone.

## 9. Privacy principle

The PoC shall distinguish between:

raw sensor measurements;

derived device events;

Edge decisions.

Where possible, the Device → Edge interface should transmit the minimum information required for the Edge decision rather than continuously transmitting unnecessary raw data.

This principle will be evaluated quantitatively in a later KPI phase.

## 10. First vertical-slice acceptance test

The first complete SSP demonstration shall satisfy:

Physical movement
       ↓
Accelerometer
       ↓
nRF52840 firmware
       ↓
Motion detection
       ↓
MOTION_EVENT
       ↓
BLE notification
       ↓
Android Edge
       ↓
Packet validation
       ↓
NORMAL → ELEVATED
       ↓
Event/state visible to user

Pass criteria

The test passes when:

The device detects the defined movement.

A valid SSP event is generated.

The event is transmitted over BLE.

Android receives the event.

Android validates the event successfully.

The Edge changes from NORMAL to ELEVATED.

The resulting state/event is visible in the Android application.

The complete sequence can be repeated reliably.

## 11. Initial KPIs

The first implementation shall establish measurement mechanisms for:

Movement detection latency

Device → Edge latency

BLE packet/notification rate

Packet loss rate

False-positive rate

False-negative rate

Device sampling rate

BLE payload size

Device/Edge processing time

Energy consumption, battery autonomy, ML performance, cloud latency, scalability, security and privacy KPIs will be added as the corresponding system layers are implemented.

## 12. Explicitly deferred functionality

The following are not required for M1/M2:

GNSS positioning

Geofencing

Cellular communication

Cloud backend

Database

Dashboard

Machine learning

Predictive models

Advanced risk scoring

Multi-device orchestration

Production security architecture

Production battery optimization

These remain part of the broader SSP roadmap.

## 13. M1 exit criterion

M1 is complete when the team has agreed on:

The physical event being demonstrated, the initial Device → Edge information flow, the logical event model, the initial Edge state model, and the measurable acceptance criteria.

The next implementation milestone is therefore:

M2 — Accelerometer/IMU integration and validated device-side motion acquisition.
