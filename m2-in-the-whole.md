# 1. The overall software development roadmap

Our system is SmartSecurePerimeter (SSP), an IoT solution that detects physical movement, interprets the event, and ultimately provides monitoring and risk information to users.

## 1. Device + Device-to-Edge communication

nRF52840-DK · Zephyr · IMU · BLE

- Acquire and validate sensor measurements.
- Process movement locally and generate SSP events.
- Define and implement the BLE interface.
- Transmit measurements/events to the Edge.

## 2. Edge processing

Android · Kotlin · BLE

- Scan, connect and subscribe to BLE notifications.
- Validate incoming data.
- Interpret events and maintain the SSP monitoring state.
- Prepare data for transmission to the Cloud.

## 3. Cloud and application processing

Backend · Database · API · Dashboard · ML

- Receive data from the Edge.
- Store measurements and events.
- Perform server-side processing or ML where appropriate.
- Display monitoring information to users.

## 4. End-to-end integration

- Connect all implemented layers.
- Verify the complete data path.
- Test failures, recovery and repeatability.
- Measure system-level KPIs and collect evidence.

These are the four development workstreams. They are not necessarily four completely sequential phases: some interface definitions and preparation can happen in parallel, but integration depends on the relevant interfaces being stable.

# 2. Where M2 fits

M2 belongs to Workstream 1: Device + Device-to-Edge communication. More specifically, it develops the device-side processing foundation before we implement BLE.

Our earlier M2 proposal remains valid. We just need to make its position in Workstream 1 explicit.

### M1 — Development foundation

Completed

Development environment, Git repository and Hello World build/flash test.

### M2 — Device sensing and local processing

Current milestone

Sensor abstraction, synthetic pipeline, physical IMU, motion detection and UART evidence.

### M3 — Device data model and BLE interface design

Freeze the logical event representation, GATT service and characteristic definitions, and BLE payload format.

### M4 — Device BLE implementation

Implement BLE advertising, GATT characteristics, notifications and the device-side communication path.

### M5 — Android BLE implementation

Scan, connect, discover services, subscribe to notifications, parse packets and display received events.

### M6 — Device-to-Edge vertical slice

Demonstrate physical movement → device detection → BLE notification → Android validation → NORMAL to ELEVATED.

The numbering above is a proposed continuation of the roadmap, not a change to the already completed M1. The important point is that M2 does not need BLE to pass, while the overall first device-to-edge communication workstream will need BLE before it can be considered complete.

# 3. Step-by-step development of Workstream 1

I recommend splitting Workstream 1 into seven implementation stages.

## Stage 1 — Establish the firmware architecture

Corresponds to M2.1.

We start with the existing Zephyr Hello World application and add clear boundaries between sensor acquisition and SSP processing.

Proposed structure:

text
```
device/firmware/hello_world/
├── CMakeLists.txt
├── prj.conf
└── src/
    ├── main.c
    ├── ssp/
    │   ├── ssp_types.h
    │   ├── ssp_motion.h
    │   └── ssp_motion.c
    └── sensor/
        ├── sensor_interface.h
        └── sensor_sim.c
```

Responsibilities:

- `main.c`: orchestrate acquisition, processing and diagnostics.
- `ssp_types.h`: define measurements, events and statuses.
- `ssp_motion.c`: calculate the motion metric and classify movement.
- `sensor_interface.h`: define a sensor-independent acquisition interface.
- `sensor_sim.c`: provide synthetic data until the physical IMU is integrated.

This stage establishes the architecture; it does not yet prove physical sensing.

Acceptance evidence: the expected source files exist, interfaces are consistent, and the firmware architecture is documented in Git.

## Stage 2 — Implement the synthetic sensing pipeline

Corresponds to M2.2.

We now implement the software pipeline without depending on the physical sensor.

text
```
Synthetic X/Y/Z
       ↓
Sensor interface
       ↓
Measurement record
       ↓
Timestamp + sequence
       ↓
Motion metric
       ↓
Deterministic event detector
       ↓
NONE / MOTION_EVENT
```

The logical measurement should include:

| Field | Purpose |
| --- | --- |
| `device_id` | Identifies the SSP device |
| `sequence` | Identifies the measurement order |
| `timestamp_ms` | Records device uptime at acquisition |
| `acceleration_x/y/z` | Stores acceleration on three axes |
| `motion_metric` | Stores the derived movement indicator |
| `event` | Represents `NONE` or `MOTION_EVENT` |
| `status` | Indicates valid acquisition or an error |

For the initial implementation, the candidate metric is:

$$
m=\sqrt{x^2+y^2+z^2}
$$

The threshold is configurable and provisional. The synthetic test can verify the processing logic, but only physical measurements can justify the final detection threshold.

Acceptance evidence: stationary synthetic samples produce `NONE`, movement samples produce `MOTION_EVENT`, and timestamps and sequence numbers progress as expected.

## Stage 3 — Build, flash and verify over UART

Corresponds to M2.3.

Compile and flash the firmware on the nRF52840-DK, then inspect the serial output.

For example:

text
```
SSP Device Firmware
M2 Motion Acquisition Test
Device ID: SSP-001

seq=0 time=1000 accel=(0.02,-0.01,1.00) motion=1.00 event=NONE
seq=1 time=1500 accel=(0.02,-0.01,1.00) motion=1.00 event=NONE
...
seq=9 time=5500 accel=(1.80,0.40,1.50) motion=2.36 event=MOTION_EVENT
```

These values are illustrative expected output, not evidence of an actual test run.

Acceptance evidence: successful build, successful flash, expected UART output and reproducible results from the repository.

At this point, Checkpoint A of M2 is complete if all its software criteria pass.

## Stage 4 — Integrate the physical IMU

Corresponds to M2.4 and M2.5.

Once the actual sensor is identified:

1. Record its manufacturer, part number and interface.
2. Verify power, wiring, address or chip-select and pin assignments.
3. Identify and configure its Zephyr driver.
4. Implement the physical sensor behind `sensor_interface.h`.
5. Verify real X/Y/Z measurements over UART.
6. Test stationary, rotated and deliberately moved positions.

The application and motion-processing logic should remain independent of the particular sensor.

Acceptance evidence: documented hardware configuration and real sensor readings that respond appropriately to physical orientation and movement.

## Stage 5 — Calibrate the motion detector

Corresponds to M2.6.

Use real measurements to establish a repeatable detection rule.

Record at least:

- Actual sampling rate and measurement units.
- Stationary noise and baseline variation.
- Response to deliberate movements.
- Detection latency.
- False detections during stationary tests.
- Missed detections during movement tests.

A useful refinement is to distinguish acceleration magnitude from deviation relative to gravity. A magnitude-only threshold can be insufficient for certain movement patterns, so the final metric should be selected from observed sensor behavior.

Acceptance evidence: a documented threshold, a repeatable test procedure and measured detection results.

## Stage 6 — Freeze the device-side interface

Corresponds to M2.7 and prepares M3.

Document the validated measurement model, units, sampling rate, timestamp semantics, sequence behavior, motion metric and event definitions.

The outcome is a stable logical contract that the subsequent BLE protocol can encode.

We should not freeze the BLE byte layout before agreeing on the data types and semantics. Equally, we should not treat the final sensor threshold as validated until the physical test results support it.

Acceptance evidence: updated interface documentation and a reproducible implementation committed to Git.

## Stage 7 — Implement BLE communication

Corresponds to M4, after the M3 interface design.

With M2 complete and the data model defined, we can add the communication functionality.

Device-side work:

- Configure BLE advertising.
- Define the SSP GATT service and characteristics.
- Expose device identity/status and event data.
- Enable notifications for the relevant characteristic.
- Encode the validated event model into the agreed BLE payload.
- Handle connection state and notification delivery.

Edge-side work begins in M5:

- Scan for the SSP device.
- Connect and discover GATT services.
- Enable notifications.
- Parse and validate the payload.
- Display received events.

Acceptance evidence: Android repeatedly receives valid BLE events generated by actual physical movement. This is the M6 vertical-slice demonstration.

# 4. How the four guidelines map to the M2 plan

| High-level guideline | Relevant milestones | Main output |
| --- | --- | --- |
| 1. Device + Device-to-Edge | M2–M6 | Physical sensing, local motion detection, BLE interface and device-to-Android event delivery |
| 2. Edge processing | M5 onward | Android packet validation, event interpretation and SSP state management |
| 3. Cloud/app processing | After the initial Edge path is stable | Backend, database, dashboard and any selected ML functionality |
| 4. End-to-end integration | After individual interfaces are available | Verified movement-to-dashboard path, system tests and KPI evidence |

There is one important nuance: the device-side processing is not the same as the Edge-side processing.

- Device: interprets accelerometer measurements and produces a `MOTION_EVENT`.
- Edge: validates that event, interprets it in the monitoring context and updates the SSP state.
- Cloud/app: stores and presents information and performs any designated server-side processing.

This separation gives each layer a clear responsibility and avoids duplicating the same logic unnecessarily.

# 5. What we should do now

Given that M1 is complete, I recommend this immediate sequence:

1. Create the M2 source structure without disturbing the validated Hello World build.
2. Implement the logical data model and motion-processing functions. Keep the threshold configurable rather than treating `1.20` as a final value.
3. Implement the synthetic sensor and feed it through the same pipeline intended for the physical IMU.
4. Connect the modules through `main.c`, including sequence numbers, uptime timestamps and UART diagnostics.
5. Update CMake and configuration, then build in a separate directory such as `/tmp/ssp-m2-test`.
6. Review the build output before flashing. Once the build succeeds, flash and verify the output, then commit the working implementation to Git.
7. Integrate the actual IMU when its exact hardware identity and connections are known.

The first programming checkpoint is therefore the synthetic pipeline, not BLE. We should leave the existing Hello World application available as a known-good reference and preserve the repository's current validation scripts until we have verified what they expect.

Our immediate objective: achieve M2 Checkpoint A, then progress to physical sensor validation and M2 Checkpoint B. Only after that should we freeze the device interface and proceed with BLE protocol implementation.

That approach follows your four high-level guidelines while keeping the M2 acceptance criteria measurable and avoiding premature work on hardware-specific drivers or communication protocols.
