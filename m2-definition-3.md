## Proposed M2 — Accelerometer Integration and Device-Side Motion Detection

| Sub-milestone | Scope | Completion evidence |
| --- | --- | --- |
| M2.1 | Firmware architecture and data model | Sensor abstraction, measurement structure, event definitions |
| M2.2 | Synthetic sensor pipeline | Timestamp, sequence, motion metric, deterministic event detection |
| M2.3 | Build, flash and UART verification | Successful build and reproducible on-board output |
| M2.4 | Identify and integrate physical IMU | Exact part number, wiring, Zephyr driver and successful sensor reads |
| M2.5 | Validate real measurements | Stationary, orientation and movement tests |
| M2.6 | Calibrate motion detection | Experimentally justified threshold and detection results |
| M2.7 | Freeze the device-side interface | Documented units, sampling rate, timestamp, sequence, metric and event representation |
| M2.8 | Final M2 acceptance test | Repeatable hardware-based motion detection with recorded evidence |

## M2 acceptance criteria

M2 will have two checkpoints.

Checkpoint A — Software pipeline validated

- [ ]Sensor-independent acquisition interface implemented.
- [ ]Synthetic X/Y/Z samples pass through the same processing pipeline intended for the real IMU.
- [ ]Sequence number and timestamp generated for each accepted measurement.
- [ ]Acceleration magnitude or another selected motion metric calculated correctly.
- [ ]Deterministic `NONE` and `MOTION_EVENT` classifications verified.
- [ ]Firmware builds, flashes and produces the expected UART output.
- [ ]Test results and implementation are committed to Git.

Checkpoint B — Physical sensor validated; required for M2 completion

- [ ]Actual IMU manufacturer, part number and interface documented.
- [ ]Hardware connections and Zephyr driver verified.
- [ ]Real acceleration measurements observed over UART.
- [ ]Stationary, rotation and deliberate-movement tests performed.
- [ ]Sampling rate and measurement units verified.
- [ ]Motion threshold calibrated using real measurements.
- [ ]Detection latency and false detections measured.
- [ ]The final hardware acceptance test passes repeatedly.
- [ ]Hardware configuration, test procedure and results documented in Git.

## Scope boundaries

M2 will not implement BLE, Android integration, GNSS, cloud services, or ML. Those belong to later milestones.

The synthetic implementation is a preparation stage, not a substitute for the physical sensor test. Also, the current proposed `sqrt(x² + y² + z²)` metric and `1.20` threshold are provisional: the threshold must be validated against real measurements before acceptance.

## Documentation and Git deliverables

The M2 milestone should ultimately reference:

- `docs/hardware/imu.md` — identified hardware and connections.
- `docs/testing/m2-test-plan.md` — repeatable test procedure and acceptance criteria.
- `docs/testing/m2-test-results.md` — actual measurements and pass/fail evidence.
- `device/firmware/hello_world/` — the existing firmware application, extended with the M2 processing pipeline unless the repository structure is deliberately changed.

The final GitHub milestone should remain open until Checkpoint B passes. Checkpoint A can be recorded as an intermediate achievement.

This structure gives us a clear, evidence-based definition of M2 without prematurely freezing implementation details that may change when the actual IMU is identified.
