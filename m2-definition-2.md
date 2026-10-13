Received. These two documents give us a much stronger basis for finalizing M2. I will treat the IOT Lab Guidelines as the external project requirements, the M1 SSP PoC Specification as the agreed functional direction, and the roadmap as the provisional implementation plan.

We should not finalize M2 yet. The next step is to compare these requirements with the implementation and testing evidence from your current development environment.

## What these documents establish

### 1. The project has two sets of acceptance criteria

- Course acceptance: the final project must cover the complete IoT pipeline, including BLE, Edge, Cloud, persistence, visualization, ML and technical reporting.
- SSP PoC acceptance: the first functional demonstration is a physical movement event detected by the nRF52840 and transmitted to Android, where it causes a `NORMAL → ELEVATED` state transition.

These are related, but they should not be conflated. The first vertical slice does not need the complete cloud or ML system to be useful.

### 2. M2 has a narrower, well-defined purpose

The current M1 specification identifies the next milestone as:

M2 — Accelerometer/IMU integration and validated device-side motion acquisition.

Its core objective is to establish a trustworthy device-side measurement and motion-detection pipeline:

nRF52840-DK + accelerometer/IMU

Raw X/Y/Z measurements

Validation, filtering and motion metric

Configurable threshold-based detection

`MOTION_EVENT` generated and testable

BLE integration will consume this output, but we should not automatically make the complete Android integration an M2 requirement. That boundary depends on the actual work already completed and how we organize the milestones.

### 3. M2 should produce measurable evidence

The eventual milestone definition should distinguish implementation from verification.

| Area | Evidence we should establish |
| --- | --- |
| Hardware | Identify the actual accelerometer/IMU and its connection to the DK or external PCB. |
| Zephyr configuration | Successful build with the required sensor support. |
| Sensor acquisition | Repeated, valid X/Y/Z measurements. |
| Measurement quality | Units, sampling interval and basic validity checks documented. |
| Motion detection | Defined metric and experimentally selected threshold. |
| Event generation | Reproducible `MOTION_EVENT` output under defined movement conditions. |
| Testing | Logs or automated tests for stationary and moving conditions. |
| Reproducibility | Build, flash and test instructions maintained in GitHub. |

The exact threshold, sampling rate and detection performance targets should be based on experiments, not invented in advance.

## What remains open

Before defining the final GitHub milestone, we need to establish:

- Which sensor hardware is actually available and connected.
- Whether the current firmware only prints a Hello World message or already includes sensor-related work.
- Whether sensor acquisition, filtering or event detection has been implemented.
- Which tests have been run and what their results demonstrate.
- Whether BLE work has started and, if so, how it should be separated from M2.

The previously successful flash-and-UART smoke test establishes a working development baseline. It does not yet establish that accelerometer acquisition works.

## Next step

Please submit the remaining documents describing the current M2 work. Useful material includes sensor datasheets or wiring notes, existing firmware source files, Zephyr configuration, build/test logs, and any preliminary sensor or BLE specifications.

I will consolidate them into a current-state assessment, identify gaps against the course requirements and M1 specification, and then propose the final M2 scope, deliverables, acceptance criteria and GitHub issues. We will keep deferred functionality—especially BLE/Android integration, cloud and ML—explicitly separated unless the evidence supports changing those boundaries.
