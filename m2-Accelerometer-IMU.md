# SmartSecurePerimeter (SSP) — M2 Specification

 ## M2 — Accelerometer/IMU Integration and Validated Device-Side Motion Acquisition

 ### 1\. M2 objective

 Integrate an external accelerometer/IMU with the nRF52840-DK running Zephyr RTOS and establish a reliable, validated device-side motion acquisition pipeline.

 The milestone shall demonstrate that the SSP Device can:

 - communicate with the physical motion sensor;
- acquire accelerometer measurements;
- validate and timestamp measurements;
- normalize sensor-specific data into an SSP device representation;
- calculate an initial motion metric;
- detect movement using a deterministic and configurable rule;
- generate a local `MOTION_EVENT`;
- expose the resulting measurements/events through the firmware's device-side interface;
- reproduce the acquisition and detection process reliably.

 M2 establishes the sensing foundation required for the later Device → BLE → Edge vertical slice.

---

 ### 2\. M2 scope

 The M2 implementation consists of:

```
Accelerometer / IMU
        ↓
Hardware interface
        ↓
Zephyr sensor driver
        ↓
Raw X/Y/Z acquisition
        ↓
Validation / normalization
        ↓
Timestamp + sequence
        ↓
Motion metric
        ↓
Deterministic motion detection
        ↓
MOTION_EVENT
```

 The nRF52840-DK is the processing platform.

 The exact sensor interface will depend on the physical IMU selected by the laboratory.

 Possible interfaces include:

 - I²C
- SPI

 The exact sensor model, interface, wiring and Zephyr driver shall be documented once the laboratory hardware is identified.

---

 ### 3\. Hardware

 Initial M2 hardware:

 - nRF52840-DK
- External accelerometer/IMU
- Required sensor breakout/adaptor hardware
- Required jumper/Dupont wiring
- Development laptop

 The sensor must be electrically compatible with the nRF52840-DK.

 The exact pinout and voltage requirements shall be verified before connection.

---

 ### 4\. Firmware environment

 M2 shall use the established project environment:

```
Project repository
    ~/ssp-iot-poc

Zephyr workspace
    ~/zephyrproject

Zephyr
    4.5.0-rc1

Target
    nrf52840dk/nrf52840

RTOS
    Zephyr
```

 Build artifacts shall remain outside the Git repository where practical.

---

 ### 5\. Sensor communication

 The nRF52840 shall communicate with the IMU using the interface supported by the selected sensor.

 Conceptually:

```
IMU
 ↓
I²C / SPI
 ↓
nRF52840 peripheral
 ↓
Zephyr sensor driver
 ↓
SSP firmware
```

 M2 shall use the appropriate Zephyr-supported driver/API where available rather than implementing unnecessary sensor-specific protocol handling from scratch.

 The sensor-specific communication protocol shall remain encapsulated below the SSP data model.

---

 ### 6\. Device-side data model

 The firmware shall normalize sensor-specific measurements into a device-side representation independent of the physical IMU.

 The initial measurement shall contain at least:

```
timestamp
sequence
acceleration_x
acceleration_y
acceleration_z
motion_metric
sensor_status
```

 The exact units and numerical representation shall be documented according to the selected sensor and firmware implementation.

 The design shall avoid coupling the later BLE protocol directly to raw sensor registers or sensor-specific messages.

---

 ### 7\. Acquisition pipeline

 The initial acquisition pipeline shall be:

```
IMU
 ↓
Raw X/Y/Z
 ↓
Sensor driver
 ↓
Measurement validation
 ↓
Unit normalization
 ↓
Timestamp
 ↓
Sequence number
 ↓
Motion metric
 ↓
Motion detector
```

 The firmware shall establish a configurable sampling interval.

 The initial sampling rate shall be selected experimentally based on:

 - sensor capabilities;
- motion characteristics;
- firmware performance;
- measurement quality.

 The sampling rate shall not be permanently fixed by this specification.

---

 ### 8\. Initial motion metric

 The firmware shall derive an initial motion metric from the accelerometer measurements.

 A simple initial implementation may use acceleration magnitude:

```
magnitude = √(x² + y² + z²)
```

 or an equivalent computationally appropriate formulation.

 The purpose of M2 is not to develop a sophisticated motion classifier.

 The purpose is to establish a reproducible device-side indicator from which a deterministic motion event can be generated.

---

 ### 9\. Motion detection

 The initial detector shall be deterministic and configurable.

 Conceptually:

```
IF motion_metric > threshold
    → MOTION_EVENT
ELSE
    → NONE
```

 The threshold shall be established experimentally during M2.

 The implementation should allow the threshold and relevant timing parameters to be changed without redesigning the sensor acquisition layer.

 More sophisticated filtering, feature extraction and classification are deferred to later milestones.

---

 ### 10\. Device-side event model

 M2 shall establish the firmware-side representation of the initial SSP event:

```
MOTION_EVENT
```

 A logical event shall contain at least:

```
device_id
sequence
timestamp
event_type
motion_metric
status
```

 This is a logical representation only.

 The final BLE byte-level representation will be defined in the subsequent SSP BLE Protocol Specification.

---

 ### 11\. Validation interface

 Before introducing BLE, M2 shall provide a simple validation mechanism over the existing development/debug interface, initially UART/console.

 Example:

```
timestamp=12345
seq=42
accel=(0.12,-0.04,1.03)
motion=0.18
event=NONE

timestamp=12355
seq=43
accel=(1.82,0.31,2.14)
motion=2.83
event=MOTION_EVENT
```

 The exact output format may evolve during implementation.

 The purpose is to make sensor acquisition and event generation directly observable and testable before adding BLE complexity.

---

 ### 12\. M2 validation tests

 The following tests shall be performed.

 #### Test A — Sensor communication

 Verify that the nRF52840 can communicate reliably with the IMU.

 Expected result:

```
Sensor detected
Sensor initialized
Measurements available
```

 #### Test B — Static measurement

 Place the board in a stable position.

 Verify:

 - X/Y/Z values are continuously acquired;
- measurements are numerically valid;
- timestamps increase;
- sequence numbers increase;
- no unexpected communication errors occur.

 #### Test C — Controlled movement

 Move the board through a defined physical motion.

 Verify:

```
movement
   ↓
accelerometer change
   ↓
motion metric change
   ↓
MOTION_EVENT
```

 #### Test D — Repeatability

 Repeat the controlled movement multiple times.

 Verify that the detector behaves consistently.

 #### Test E — Idle behavior

 Leave the board stationary.

 Verify that the detector does not continuously generate motion events under normal stationary conditions.

 The exact false-positive behavior will be quantified as the implementation matures.

---

 ### 13\. Initial M2 KPIs

 M2 shall establish measurement mechanisms for:

 - Sensor sampling rate
- Measurement interval
- Sensor communication errors
- Measurement validity
- Timestamp consistency
- Sequence continuity
- Motion detection latency
- Motion-event repeatability
- Initial false-positive behavior
- Firmware processing time

 The following remain outside M2:

 - BLE latency
- BLE packet loss
- Android processing latency
- Cloud latency
- ML performance
- Battery autonomy

 These will be measured when the corresponding system layers exist.

---

 ### 14\. Privacy and architecture principle

 M2 shall establish the separation between:

```
Raw sensor measurement
        ↓
Device-derived metric
        ↓
Device event
```

 This separation is important to the SSP architecture.

 The device should eventually be able to communicate a derived event rather than continuously transmitting all raw sensor measurements.

 M2 therefore establishes the technical foundation for the later privacy-aware Device → Edge communication model.

---

 ### 15\. Explicitly deferred functionality

 The following are not required for M2:

 - GNSS/GPS integration
- Position processing
- Geofencing
- BLE GATT implementation
- Android application
- Cloud communication
- Database
- Dashboard
- Machine learning
- Predictive risk assessment
- Advanced motion classification
- Adaptive sampling
- Production power optimization
- Multi-device operation
- Production security architecture

 GNSS will be integrated as a subsequent sensing capability once the actual GNSS hardware and interface have been validated.

---

 ### 16\. M2 deliverables

 M2 shall produce:

 1. Connected and documented IMU hardware.
2. Sensor wiring/pinout documentation.
3. Zephyr sensor configuration.
4. Working device-side sensor acquisition firmware.
5. Normalized accelerometer data representation.
6. Timestamp and sequence mechanism.
7. Initial motion metric.
8. Configurable deterministic motion detector.
9. `MOTION_EVENT` generation.
10. UART/console validation output.
11. Repeatable sensor/motion test results.
12. Updated project documentation.

---

 ### 17\. M2 acceptance test

 M2 passes when:

```
Physical movement
       ↓
Accelerometer / IMU
       ↓
nRF52840
       ↓
Zephyr sensor driver
       ↓
Valid X/Y/Z measurements
       ↓
Timestamp + sequence
       ↓
Motion metric
       ↓
Deterministic motion detection
       ↓
MOTION_EVENT
       ↓
Observable UART/console output
```

 The test must be repeatable and demonstrate both:

```
stationary device → no continuous false movement
```

 and:

```
defined physical movement → reliable MOTION_EVENT
```

---

 ### 18\. M2 exit criterion

 M2 is complete when the nRF52840-DK can reliably acquire measurements from the selected accelerometer/IMU, transform them into the initial SSP device-side representation, detect the defined physical movement using a deterministic rule, and demonstrate the resulting `MOTION_EVENT` repeatedly through the device validation interface.

 The next milestone is:

 **M3 — SSP Device Data Model + BLE Protocol + Device BLE communication.**

 This gives us a clean boundary: **M2 proves that the physical device can sense and interpret motion; M3 turns that validated device information into a formal communication interface toward the Edge.**
