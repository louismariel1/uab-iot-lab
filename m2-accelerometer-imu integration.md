## M2 — Accelerometer/IMU Integration & Device-Side Motion Acquisition

 The goal of M2 should be very specific:

 > **The nRF52840-DK reliably acquires real accelerometer data, timestamps it, performs basic validation/processing, and produces a repeatable motion-detection result over UART.**

 We should **not introduce BLE yet**. M2 should prove the sensing and local-processing side independently.

 ### M2 roadmap

```
M2.1  Identify IMU hardware
        ↓
M2.2  Confirm electrical/interface requirements
        ↓
M2.3  Select Zephyr sensor driver
        ↓
M2.4  Build minimal sensor firmware
        ↓
M2.5  Verify raw X/Y/Z measurements
        ↓
M2.6  Add timestamp + sequence
        ↓
M2.7  Add basic validation/filtering
        ↓
M2.8  Implement motion metric
        ↓
M2.9  Implement deterministic MOTION_EVENT
        ↓
M2.10 Characterize detection experimentally
        ↓
M2.11 Freeze device-side sensor interface
        ↓
M2 EXIT TEST
```

 ## M2.1 — Identify the actual IMU

 **First task: establish exactly which accelerometer/IMU hardware we have.**

 We need:

 - Manufacturer
- Part number
- Breakout/evaluation board
- Interface: I²C or SPI
- Supply voltage
- Accelerometer range
- Configurable output data rate
- Physical connection to the nRF52840-DK

 If the exact IMU is already available, we should use it rather than selecting a hypothetical sensor.

 **Deliverable:**

```
docs/hardware/imu.md
```

 containing the exact hardware and connection information.

---

 ## M2.2 — Connect and electrically verify it

 Before writing firmware:

```
nRF52840-DK
     │
     ├── VCC
     ├── GND
     ├── SDA/SCL       (I²C)
     │       or
     └── MOSI/MISO/SCK/CS (SPI)
             │
             ▼
            IMU
```

 Verify:

 - Power
- Ground
- Bus connections
- Address/chip-select
- No conflicting peripherals
- Appropriate voltage levels

 Don't troubleshoot Zephyr and wiring simultaneously if we can avoid it.

---

 ## M2.3 — Establish the Zephyr sensor path

 The preferred architecture is:

```
IMU hardware
     ↓
Zephyr sensor driver
     ↓
Zephyr sensor API
     ↓
SSP application
```

 Rather than writing a custom driver immediately.

 We should determine whether the selected IMU already has a suitable Zephyr driver.

 The firmware should remain structured approximately as:

```
main.c
   │
   ▼
SSP sensor acquisition
   │
   ▼
Zephyr sensor API
   │
   ▼
IMU driver
   │
   ▼
hardware
```

---

 ## M2.4 — Minimal sensor firmware

 Create a new application rather than modifying our validated Hello World:

```
device/firmware/
├── hello_world/
└── sensor_test/
```

 The initial application should do **only this**:

```
initialize IMU
      ↓
read X/Y/Z
      ↓
print values
      ↓
delay
      ↓
repeat
```

 Example output conceptually:

```
IMU initialized
ACCEL x=0.02 y=-0.01 z=0.98
ACCEL x=0.03 y=-0.02 z=0.99
ACCEL x=0.15 y=0.04 z=1.02
```

 The exact units depend on the sensor/driver.

---

 ## M2.5 — Validate raw measurements

 This is an important checkpoint.

 We should physically test:

 ### Stationary

 Place the board still.

 Expected behavior:

 - X approximately stable
- Y approximately stable
- Z approximately reflects gravity

 ### Rotate board

 Rotate it through different orientations.

 Expected behavior:

```
orientation changes
       ↓
distribution of X/Y/Z changes
```

 ### Move board

 Move/shake the board.

 Expected behavior:

```
X/Y/Z change dynamically
```

 This proves that we're reading the **actual physical sensor**, not merely receiving plausible-looking values.

---

 ## M2.6 — Add timestamp and sequence

 Once raw acquisition works, produce an internal measurement record:

```
measurement {
    timestamp
    sequence
    accel_x
    accel_y
    accel_z
}
```

 For example:

```
SEQ=101 TIME=123456
X=...
Y=...
Z=...
```

 This is important because the same structure will eventually become the foundation for the BLE protocol.

---

 ## M2.7 — Basic validation and filtering

 Keep this deliberately simple.

 Possible checks:

```
invalid sensor reading?
       ↓
discard / flag

valid reading?
       ↓
accept
```

 Then introduce minimal filtering if measurements are noisy.

 We should **measure the raw signal first** rather than blindly adding filters.

 The purpose is to determine:

 - Sensor noise
- Measurement stability
- Sampling behavior
- Appropriate motion threshold

---

 ## M2.8 — Define the motion metric

 We need a deterministic quantity that represents movement.

 A natural first candidate is acceleration magnitude:

```
                 ______________________
magnitude = √(x² + y² + z²)
```

 Because gravity is included, a stationary device should have a magnitude approximately around **1 g**.

 For motion detection, we could therefore initially examine:

```
|magnitude - 1g|
```

 or another experimentally justified metric.

 The important point is:

 > **Do not choose the final threshold before collecting real measurements.**

---

 ## M2.9 — Implement `MOTION_EVENT`

 Once the signal is understood:

```
measurement
     ↓
motion metric
     ↓
threshold / rule
     ↓
MOTION_EVENT
```

 For example conceptually:

```
IF motion_metric > threshold
    → MOTION_EVENT
```

 The implementation should initially be deterministic and explainable.

 No ML.

---

 ## M2.10 — Experimental characterization

 We should create a small repeatable test procedure.

 For example:

```
Test A — stationary
Test B — gentle movement
Test C — deliberate movement
Test D — short movement
Test E — sustained movement
```

 Record:

 - Sampling rate
- Raw values
- Motion metric
- Detection latency
- Detected events
- False detections

 This gives us actual evidence for selecting the motion threshold.

---

 ## M2.11 — Freeze the device-side interface

 At the end of M2 we should know:

```
Sensor:
    exact device

Interface:
    I²C/SPI

Sampling:
    X Hz

Data:
    X/Y/Z

Timestamp:
    available

Sequence:
    available

Motion metric:
    defined

Detection rule:
    defined

Event:
    MOTION_EVENT
```

 This becomes the input to **M3/M4 BLE protocol development**.

---

 # M2 exit test

 The final M2 test should look like this:

```
                 ┌──────────────┐
                 │ nRF52840-DK  │
                 └──────┬───────┘
                        │
                        ▼
                     IMU
                        │
                        ▼
                  X / Y / Z
                        │
                        ▼
                  timestamp
                  + sequence
                        │
                        ▼
                 motion metric
                        │
                        ▼
                deterministic rule
                        │
              ┌─────────┴─────────┐
              │                   │
           NORMAL            MOTION_EVENT
```

 And the UART evidence should demonstrate the complete sequence.

 ### M2 is DONE when

 - [ ] Exact IMU identified.
- [ ] Hardware connected and verified.
- [ ] Zephyr driver successfully communicates with IMU.
- [ ] Real X/Y/Z measurements observed.
- [ ] Stationary/rotation/movement tests behave as expected.
- [ ] Timestamp and sequence implemented.
- [ ] Sampling rate measured.
- [ ] Motion metric defined from actual measurements.
- [ ] Deterministic `MOTION_EVENT` detection implemented.
- [ ] Detection behavior experimentally characterized.
- [ ] Results documented.
- [ ] Sensor acquisition is reproducible from the Git repository.

 ### One important constraint

 I would **not proceed to BLE until M2 passes**.

 The resulting progression should be:

```
M0  Firmware environment       ✓
M1  PoC specification          ✓
M2  Sensor + motion           ← NOW
M3  Device data model
M4  BLE protocol
M5  Android BLE
M6  Device → Edge vertical slice
```

 That gives us a very clean engineering progression: **first prove that the device can sense reality, then prove that it can communicate what it sensed.**
