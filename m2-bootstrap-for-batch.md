Below is a self-contained project bootstrap document designed to be saved in your repository and pasted into a new ChatGPT session. It captures the project objective, confirmed achievements, repository layout, development strategy, verification requirements, unresolved decisions, and the immediate next action.

I recommend saving it as `docs/architecture/SSP_PROJECT_BOOTSTRAP.md`. It deliberately distinguishes verified facts from proposed work so a future session does not mistake a plan for an implementation that already exists.

Writing

# SmartSecurePerimeter (SSP) — Project Bootstrap and Continuation Guide

**Document purpose:** Preserve project context across ChatGPT sessions and provide an actionable continuation plan.

**Project:** SmartSecurePerimeter (SSP)\
**Repository:** `~/ssp-iot-poc`\
**Development environment:** Windows + WSL 2 Ubuntu\
**Firmware platform:** Zephyr RTOS on nRF52840\
**Edge platform:** Android / Kotlin\
**Current strategic priority:** Implement a working BLE Device → Edge communication path using YAML-driven code generation and automated verification.

**Status:** Hello World hardware baseline previously verified. BLE implementation remains unverified until the repository inventory confirms the actual files and subsequent tests establish functionality.

---

# 1\. Instructions for the next ChatGPT session

Continue the existing SSP project. Do not restart the architectural discussion or assume proposed code already exists.

First, read this bootstrap document. Then ask the user to provide the latest repository inventory or terminal output if it is not already available.

Follow these rules:

1. Preserve the existing working Hello World firmware and its two successful test scripts.
2. Do not create another audit script merely to inspect the repository.
3. Verify the actual files before modifying or generating code.
4. Prefer coherent batches of generated code over manual, file-by-file implementation.
5. Use one authoritative YAML protocol contract to generate C definitions, Kotlin definitions, documentation and shared test vectors.
6. Verify each implementation batch with automated tests before proceeding to the next dependency boundary.
7. Distinguish software build success from physical hardware integration success.
8. Do not claim a test passed unless its output or other reliable evidence confirms the result.
9. Do not start cloud, database, dashboard, ML or production security work before the first Device → Edge vertical slice works.
10. Keep synthetic event generation available so BLE can be developed before the physical IMU integration is complete.

**Immediate task:** Establish the real repository baseline, then implement and verify Batch A: the BLE protocol contract, generator/templates and shared protocol test vectors.

---

# 2\. Project objective

SSP is an IoT monitoring proof of concept that demonstrates distributed intelligence between a physical device and an Android Edge application.

The first complete demonstration must follow this sequence:

Physical movement → Accelerometer/IMU → nRF52840 firmware → Local motion detection → `MOTION_EVENT` → BLE GATT notification → Android BLE client → Packet validation → SSP state changes from `NORMAL` to `ELEVATED` → Event and state visible to the user

The initial PoC is intentionally deterministic. It does not require machine learning, geofencing, cloud processing or advanced risk prediction.

The primary objective is to demonstrate a repeatable Device → Edge architecture, not to build every planned SSP feature at once.

## 2.1 Device responsibilities

- Acquire accelerometer measurements.
- Validate and process measurements locally.
- Detect movement using a configurable deterministic rule.
- Generate SSP events.
- Advertise and communicate over BLE.
- Expose device status and event data through a GATT service.

## 2.2 Edge responsibilities

- Scan for the SSP device.
- Connect over BLE and discover the SSP service.
- Subscribe to notifications.
- Decode and validate incoming packets.
- Validate sequence and timestamp behavior.
- Change the SSP monitoring state in response to valid events.
- Display the connection, received events and current state.

## 2.3 Deferred responsibilities

The following are outside the first vertical slice:

- Cloud backend and database.
- Dashboard and web frontend.
- Machine learning and predictive risk analysis.
- GNSS and geofencing.
- Cellular connectivity.
- Production-scale multi-device management.
- Production security and battery optimization.
- Advanced adaptive monitoring policies.

These can be introduced after the Device → Edge path is reliable.

---

# 3\. What has already been achieved

## 3.1 Repository and development environment

The repository exists at `~/ssp-iot-poc`.

The previously reported repository layout is:

```
ssp-iot-poc/
├── cloud/
│   ├── backend/
│   ├── database/
│   └── frontend/
├── device/
│   ├── drivers/
│   ├── firmware/
│   │   └── hello_world/
│   │       ├── CMakeLists.txt
│   │       ├── prj.conf
│   │       └── src/main.c
│   ├── protocol/
│   └── tests/
├── docs/
│   ├── architecture/
│   ├── protocol/
│   ├── reports/
│   └── testing/
├── edge/
│   ├── protocol/
│   └── tests/
├── ml/
└── tools/
    ├── compile_and_flash_test.sh
    └── flash_and_uart_smoke_test.sh
```

This is the last confirmed tree provided by the user, not a guarantee that the repository has not changed since then.

## 3.2 Known-good firmware

The existing application is located at:

`device/firmware/hello_world/`

Its intended target is:

`nrf52840dk/nrf52840`

The existing build directory is:

`/tmp/ssp-hello-world`

The environment previously reported:

- Python 3.12.15.
- pyenv virtual environment `zephyr-3.12`.
- West 1.5.0.
- pyserial 3.5.
- Zephyr workspace at `~/zephyrproject`.
- Zephyr build identity reported by the firmware: `v4.5.0-rc1-43-ga90cd40a5f0a`.
- SEGGER J-Link 9.82.
- Serial device `/dev/ttyACM0`.

The reported Zephyr build identity is evidence of the firmware image's identity. Verify the currently installed workspace revision separately when necessary.

## 3.3 Successful test 1: flash and UART smoke test

Script:

`tools/flash_and_uart_smoke_test.sh`

Authoritative underlying test:

`~/flash_hello_test.sh`

Reported behavior:

1. Confirm Windows USB preparation.
2. Activate or verify the Python environment.
3. Verify the Zephyr workspace.
4. Verify UART availability.
5. Check the Hello World HEX and ELF files.
6. Execute the authoritative flash/UART test.
7. Confirm the expected UART message.

The reported output confirmed successful flashing using the J-Link runner and receipt of:

```
SSP PoC Hello World! nRF52840
```

The final result was:

```
SSP FLASH AND UART TEST: PASS
```

## 3.4 Successful test 2: compile, flash and UART

Script:

`tools/compile_and_flash_test.sh`

Reported behavior:

1. Confirm Windows USB preparation.
2. Verify the Python environment and tools.
3. Verify the SSP firmware sources.
4. Enter the Zephyr workspace.
5. Verify UART availability.
6. Perform a pristine Zephyr build.
7. Verify the HEX and ELF artifacts.
8. Run the existing flash/UART test.
9. Report success only after the test succeeds.

The reported fresh build completed successfully. The output identified:

```
Board: nrf52840dk/nrf52840
Zephyr: 4.5.0-rc1
```

The expected UART message was received and the final result was:

```
SSP COMPILE, FLASH AND UART: PASS
```

**Important:** These are previously successful tests, not evidence that BLE works. Their results apply to the tested versions and conditions.

Keep both scripts and the Hello World application unchanged during the initial BLE work.

---

# 4\. Current development stage

The project is transitioning from its validated firmware baseline to the first BLE implementation.

The selected development method is:

**YAML protocol contract → templates and generator → shared test vectors → Zephyr BLE peripheral → Android BLE client → automated integration.**

The repository's actual BLE implementation status has not yet been established conclusively.

Previous proposed inventories identified these as expected deliverables:

- `protocol/ssp_ble_v1.yaml`
- `tools/ble_codegen/`
- `tests/ble/`
- `device/firmware/ble_peripheral/`
- `edge/android/`
- `tools/build_ble_demo.sh`

Do not assume these files exist. Confirm their presence and inspect their content before using or replacing them.

A previously reported untracked file, `tools/verification.sh`, was also mentioned. Its current existence and contents are not confirmed. Do not depend on it until the repository inventory verifies it.

## Current objective

Establish the real repository baseline, then implement Batch A and proceed quickly toward a working synthetic-event BLE demonstration.

Do not start physical IMU calibration before the communication architecture is demonstrated. The sensor can be integrated behind a stable event-producer interface later.

---

# 5\. Target architecture

## 5.1 Logical architecture

```
DEVICE — nRF52840 / Zephyr
┌──────────────────────────────┐
│ Synthetic event producer     │
│             or                │
│ Physical IMU acquisition     │
├──────────────────────────────┤
│ Motion/event processing      │
├──────────────────────────────┤
│ SSP event model              │
├──────────────────────────────┤
│ Generated protocol encoder   │
├──────────────────────────────┤
│ BLE GATT service             │
│ Advertising + notifications  │
└──────────────┬───────────────┘
               │ BLE
               ▼
EDGE — Android / Kotlin
┌──────────────────────────────┐
│ BLE scan/connect/discover    │
├──────────────────────────────┤
│ Notification subscription    │
├──────────────────────────────┤
│ Generated protocol decoder   │
├──────────────────────────────┤
│ Packet/sequence validation   │
├──────────────────────────────┤
│ SSP state reducer            │
├──────────────────────────────┤
│ Monitoring UI and event log  │
└──────────────────────────────┘
```

## 5.2 Proposed repository structure

The final structure should be based on the existing repository and actual implementation needs. The following is the target, not a claim that all files exist.

```
ssp-iot-poc/
├── protocol/
│   └── ssp_ble_v1.yaml
├── tools/
│   ├── ble_codegen/
│   │   ├── requirements.txt
│   │   ├── generate.py
│   │   └── templates/
│   │       ├── ssp_protocol.h.j2
│   │       ├── SspProtocol.kt.j2
│   │       └── protocol.md.j2
│   ├── build_ble_demo.sh
│   ├── compile_and_flash_test.sh
│   └── flash_and_uart_smoke_test.sh
├── tests/
│   └── ble/
│       ├── test_vectors.json
│       └── test_protocol.py
├── device/
│   └── firmware/
│       ├── hello_world/             # Preserve
│       └── ble_peripheral/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               ├── main.c
│               ├── ssp_gatt.c
│               ├── ssp_event.c
│               ├── ssp_event.h
│               ├── ssp_advertising.c
│               └── generated/
│                   └── ssp_protocol.h
├── edge/
│   ├── protocol/
│   └── android/
│       ├── settings.gradle.kts
│       ├── build.gradle.kts
│       ├── gradlew
│       └── app/
│           ├── build.gradle.kts
│           └── src/
│               ├── main/
│               │   ├── AndroidManifest.xml
│               │   └── java/org/ssp/
│               │       ├── protocol/SspProtocol.kt
│               │       ├── ble/SspBleClient.kt
│               │       ├── ble/AndroidSspBleClient.kt
│               │       ├── state/SspState.kt
│               │       └── ui/
│               └── test/
├── docs/
│   ├── architecture/
│   │   └── SSP_PROJECT_BOOTSTRAP.md
│   ├── protocol/
│   │   └── ssp-ble-v1.md
│   └── testing/
│       └── ssp-ble-integration.md
└── ...
```

Avoid creating redundant modules merely to match this layout. The generated files and the handwritten modules must have clear ownership.

---

# 6\. Why YAML + templates + automated builds

The project must avoid generating and debugging each source file independently.

Manual, script-by-script implementation introduces repeated definitions of UUIDs, field sizes, event codes, byte order, and validation rules. These definitions can drift between firmware, Android and documentation.

The chosen method centralizes the contract and automates the repetitive implementation.

## 6.1 One authoritative definition

Recommended canonical path:

`protocol/ssp_ble_v1.yaml`

The YAML should define at least:

- Protocol name and version.
- GATT service UUID.
- Characteristic UUIDs.
- Characteristic properties.
- Payload fields and their types.
- Field offsets and sizes, or enough information to derive them unambiguously.
- Byte order.
- Event and status enumerations.
- Device identifier representation.
- Sequence and timestamp semantics.
- Units and range constraints.
- Reserved fields and their required values, if any.
- Shared test-vector references or definitions.

The exact contract must be reviewed and frozen before generating dependent implementations.

## 6.2 Generator and templates

The generator should validate the YAML and produce:

- C protocol definitions and encoding/decoding helpers.
- Kotlin protocol definitions and packet decoding.
- Protocol documentation.
- Shared test-vector artifacts where appropriate.

Templates should contain formatting and code-generation logic. They must not independently hardcode protocol facts that belong in YAML.

Generated code should be clearly marked as generated and should not be edited manually.

## 6.3 Required generator operations

Implement three explicit operations:

- `validate`: validate the contract without writing generated files.
- `generate`: render and write generated artifacts.
- `check`: render the expected artifacts in memory and compare them against the existing generated files, returning failure if anything is missing or stale.

A check that only renders templates without comparing their output to existing files is insufficient.

Generation must be deterministic: identical YAML and generator/template versions should produce identical output.

## 6.4 Shared test vectors

Use one shared test-vector source to verify the C and Kotlin implementations.

Test valid packets and invalid packets, including:

- Incorrect payload length.
- Unsupported protocol version.
- Unknown event value.
- Invalid status.
- Invalid or reserved field values.
- Boundary values for integer fields.
- Endianness errors.
- Sequence wraparound behavior where applicable.

Python tests can validate the contract, vector integrity and generator behavior. Kotlin tests must validate the Android decoder. Firmware compilation must validate C integration; target-specific behavior still requires hardware tests.

---

# 7\. Protocol design issues to preserve

A previously proposed initial event payload is 17 bytes.

| Offset | Field | Size |
| --- | --- | --- |
| 0 | Version | 1 byte |
| 1 | Event type | 1 byte |
| 2 | Status | 1 byte |
| 3–6 | Device ID | 4 bytes |
| 7–10 | Sequence | 4 bytes |
| 11–14 | Timestamp | 4 bytes |
| 15–16 | Motion metric | 2 bytes |

Total: 17 bytes.

A previously proposed valid vector was:

```
01 01 00 01 50 53 53 01 00 00 00 E8 03 00 00 08 07
```

This vector encodes the following under the previously proposed layout:

- Version: `1`.
- Event type: `1`.
- Status: `0`.
- Device ID: `0x53535001`.
- Sequence: `1`.
- Timestamp: `1000` milliseconds.
- Motion metric: `0x0708` or `1800` in little-endian representation.

The integer `0x53535001` is decimal `1397960705`. Do not replace it with the incorrect decimal value `1397968897`.

**Important qualification:** This is a proposed protocol layout and vector, not a confirmed repository artifact or frozen specification. Verify it against the actual YAML and code before treating it as authoritative.

The field `motion_mg` also needs a precise definition. A synthetic value of 1800 is useful for transport testing, but it must not be presented as a validated physical measurement until the sensor algorithm and units are established.

## 7.1 Sequence numbers

Do not implement sequence validation using a simple `sequence <= previousSequence` rule.

A robust implementation must consider:

- 32-bit sequence wraparound.
- Reconnection.
- Device reboot and sequence reset.
- Duplicate notifications.
- Missing sequence numbers.
- Whether a sequence is maintained per event or per measurement.

For the first PoC, define and test a deterministic policy. Avoid treating every reset as a protocol failure without a way to establish a new session.

## 7.2 UUID generation

Generate C UUID definitions and Kotlin UUID constants from YAML. Do not independently hardcode the same values in multiple implementation files.

Verify UUID format, uniqueness and consistency between advertising, GATT definitions and Android service discovery.

## 7.3 GATT responsibilities

The Zephyr implementation must explicitly implement:

- Connectable advertising.
- Service and characteristic registration.
- Device identification/status read behavior.
- Event characteristic notification support.
- Notification subscription tracking.
- Connection and disconnection handling.
- Safe notification delivery only when a connection and subscription exist.
- Synthetic event production for initial integration tests.

The GATT subscription logic must reference the actual characteristic attribute and connection. A `NULL` subscription attribute or an undefined global attribute is not acceptable.

Advertising must be started after successful Bluetooth initialization, with failures propagated to the application rather than silently ignored.

---

# 8\. Execution plan: four implementation batches

The plan uses four batches. Each batch is large enough to reduce repetitive work but bounded by a testable acceptance gate.

## Batch A — Protocol contract, generator and shared vectors

**Scope:** Steps 1–4.

Deliverables:

1. Canonical YAML contract.
2. Schema and semantic validation.
3. Generator dependencies and entry point.
4. C, Kotlin and documentation templates.
5. Generated C and Kotlin protocol artifacts.
6. Shared test vectors.
7. Python protocol/generator tests.
8. Deterministic generation and stale-file detection.

Acceptance criteria:

- YAML validation succeeds.
- Invalid definitions are rejected.
- UUIDs are unique and consistent.
- Payload fields, offsets and sizes agree.
- Generated files contain no independently maintained protocol constants.
- Repeated generation produces identical output.
- `check` fails when generated output is stale.
- Shared vectors agree with the documented wire format.
- Python tests pass.

Do not proceed to firmware implementation until the protocol and generator are internally consistent.

## Batch B — Zephyr BLE peripheral

**Scope:** Steps 5–7.

Deliverables:

1. Separate firmware application under `device/firmware/ble_peripheral/`.
2. Zephyr configuration and CMake integration.
3. Generated C protocol definitions.
4. GATT service and characteristics.
5. Connectable advertising.
6. Connection and subscription management.
7. Synthetic event producer.
8. Notification encoder and delivery.
9. UART diagnostics for connection and event activity.

Build directory:

`/tmp/ssp-ble-demo`

The BLE build must not reuse `/tmp/ssp-hello-world`.

Acceptance criteria:

- Clean Zephyr build succeeds.
- The firmware links the generated protocol definitions.
- Advertising is implemented.
- The service and characteristics compile correctly.
- Status reads return a defined payload.
- Notifications are attempted only when a connected client has subscribed.
- Synthetic events use the shared wire-format definition.
- A BLE client can discover the service and receive correctly encoded notifications on hardware.

A firmware build alone does not satisfy the hardware acceptance criterion.

## Batch C — Android BLE client and state model

**Scope:** Steps 8–10.

Deliverables:

1. Minimal Android Gradle project.
2. Manifest and BLE runtime permissions.
3. BLE scan/connect/discovery implementation.
4. Notification subscription.
5. Generated Kotlin protocol decoder.
6. Sequence and timestamp validation.
7. SSP state reducer.
8. Monitoring UI with connection state, events and current state.
9. Kotlin unit tests.

Initial state behavior:

```
NORMAL
  |
  | valid MOTION_EVENT
  v
ELEVATED
```

Invalid packets must not trigger a state transition.

The BLE lifecycle must handle permission denial, scan failures, connection loss, failed discovery, missing characteristics and reconnection.

Acceptance criteria:

- Android project builds.
- Decoder tests pass against shared vectors.
- Invalid packets are rejected.
- Valid events update the state reducer.
- The UI shows connection state, received events and the SSP state.
- The application subscribes to the actual event characteristic.

Do not report the Android integration as complete if tests were skipped.

## Batch D — Build orchestration, integration and evidence

**Scope:** Steps 11–13.

Deliverables:

- `tools/build_ble_demo.sh`
- `docs/testing/ssp-ble-integration.md`
- Integration test procedure or automation.
- Recorded integration results.
- Reproducible commit.

The orchestrator should perform, in dependency order:

1. Validate YAML.
2. Generate code.
3. Verify generated-code freshness.
4. Run Python protocol and generator tests.
5. Run Kotlin unit tests.
6. Build the Zephyr BLE application.
7. Report whether Android build/tests ran or were skipped.
8. Optionally execute hardware integration when explicitly requested and the required hardware is available.

The command must fail if any mandatory stage fails. It must never print a full-system PASS after silently skipping Android tests or hardware checks.

Software-only and hardware-integration modes must be clearly distinguished.

Acceptance criteria:

- Repeated generation is reproducible.
- All mandatory automated tests pass.
- The firmware build passes.
- Android build and tests pass.
- Hardware integration evidence records actual received notification bytes.
- The Android application displays `NORMAL → ELEVATED`.
- The demonstration can be repeated reliably.
- Results and the relevant Git revision are documented.

---

# 9\. Repository verification workflow

The initial verification plan uses these statuses:

- `PASS`: required evidence confirms the check succeeded.
- `FAIL`: the check ran and found a defect.
- `NOT PRESENT`: the expected artifact does not exist.
- `BLOCKED`: the check cannot run because a dependency or prerequisite is unavailable.
- `NOT VERIFIED`: evidence has not yet been collected.

Do not mark a module verified solely because the file exists.

For each relevant artifact, record:

- Existence.
- Provenance: generated, handwritten or modified.
- Static correctness.
- Automated test/build evidence.
- Hardware integration evidence, where applicable.

Use repository-wide inventory commands before assuming the generated files are absent.

A suitable initial read-only inspection is:

```
cd ~/ssp-iot-poc

echo "=== Git status ==="
git status --short

echo "=== Last commit ==="
git log -1 --oneline

echo "=== Tracked files ==="
git ls-files

echo "=== All repository files ==="
find . -path './.git' -prune -o -type f -print | sort

echo "=== BLE and generator candidates ==="
find device edge tools tests protocol docs \
  -type f \
  \( -iname '*ble*' \
     -o -iname '*protocol*' \
     -o -iname '*generate*' \
     -o -iname '*codegen*' \
     -o -iname '*vector*' \
     -iname '*gradle*' \
     -o -iname '*.yaml' \
     -o -iname '*.yml' \
     -o -iname '*.j2' \
     -o -iname '*.kt' \
     -o -iname '*.java' \
     -o -iname '*.c' \
     -o -iname '*.h' \
     -o -iname '*.py' \
     -o -iname '*.json' \) \
  -print 2>/dev/null | sort
```

This is sufficient for the initial inventory. Do not add another audit script unless a concrete recurring need justifies it.

After reviewing the output, establish the actual baseline and remove ambiguity about which artifacts exist.

---

# 10\. Acceptance gates

| Gate | Required evidence |
| --- | --- |
| G0 — Inventory | Actual file list, Git state, missing artifacts and baseline identified |
| G1 — Protocol contract | YAML and payload definition validate |
| G2 — Generation | Deterministic generation and stale-file checks pass |
| G3 — Protocol | C/Kotlin implementations agree with shared valid and invalid vectors |
| G4 — Firmware | Separate Zephyr BLE build succeeds |
| G5 — Peripheral | Hardware advertises, connects, exposes GATT and sends notifications |
| G6 — Android | Application builds and implements BLE connection and subscription |
| G7 — State transition | Valid event changes `NORMAL` to `ELEVATED`; invalid event does not |
| G8 — End-to-end | Repeated device-to-Android demonstration passes with evidence |
| G9 — Reproducibility | Clean regeneration and build/test results are recorded against a Git revision |

A gate can be marked complete only when its required evidence is available.

---

# 11\. Key challenges and risks

## 11.1 Unverified generated artifacts

Earlier code examples and proposed file inventories do not prove that the corresponding files exist in Git.

**Mitigation:** inventory first; distinguish proposed code from actual code.

## 11.2 Protocol drift

Repeated constants in C, Kotlin, YAML and documentation can diverge.

**Mitigation:** one authoritative YAML contract, generated definitions and shared vectors.

## 11.3 Incomplete generator checks

A generator may render templates without verifying that checked-in outputs match.

**Mitigation:** compare expected generated content against actual files and fail on differences.

## 11.4 Zephyr API and version differences

The installed workspace must be checked before relying on sample code from another Zephyr release.

**Mitigation:** verify the workspace revision and compile the separate BLE application before flashing.

## 11.5 GATT implementation defects

Undefined attributes, incorrect subscription checks, missing callbacks and absent advertising can produce code that looks plausible but does not function.

**Mitigation:** review actual attribute definitions, callbacks, connection state and subscription state; verify behavior with a real BLE client.

## 11.6 Android project incompleteness

Protocol code alone is not an Android BLE client.

**Mitigation:** include the Gradle project, manifest, permissions, BLE lifecycle, notification subscription, decoder, state reducer and UI in Batch C.

## 11.7 Sequence and reconnect semantics

Sequence counters may wrap or reset when the device restarts.

**Mitigation:** define explicit wraparound, duplicate, missing-event and session-reset behavior.

## 11.8 Synthetic versus physical motion

Synthetic events validate communication but do not validate sensor acquisition or detection thresholds.

**Mitigation:** treat synthetic-event transport and physical sensing as separate acceptance stages.

## 11.9 Development board versus custom PCB

The previously tested target is `nrf52840dk/nrf52840`. Confirm whether the hardware under test is the Nordic development kit or a custom PCB before drawing hardware conclusions.

## 11.10 BLE security

The first PoC can focus on basic transport, but the eventual design must consider pairing, access permissions, privacy and spoofed events.

**Mitigation:** document the initial security assumptions and defer production security implementation explicitly rather than treating an unprotected demo as production-ready.

---

# 12\. Physical IMU integration after BLE

Once synthetic events are reaching Android and the state transition is repeatable, integrate the actual sensor.

The physical IMU's manufacturer, model, bus and wiring must be identified first.

The eventual pipeline should be:

```
Physical IMU
    ↓
Sensor driver
    ↓
Measurement validation
    ↓
Motion processing
    ↓
MOTION_EVENT
    ↓
Existing SSP event interface
    ↓
Generated BLE encoder
    ↓
BLE notification
    ↓
Android state update
```

The event producer should be replaceable without redesigning the BLE transport.

Measure the real sensor's sampling rate, stationary noise, motion response, detection latency and false-positive/false-negative behavior before finalizing thresholds.

The synthetic event source remains useful as a regression test after sensor integration.

---

# 13\. Immediate action list

Perform the following steps in order.

## Action 1 — Establish the repository baseline

Run the read-only inventory from Section 9 and review the results.

Identify which protocol, generator, firmware and Android files actually exist. Confirm the working-tree status and preserve any existing work.

## Action 2 — Implement Batch A

Once the inventory is established:

1. Select `protocol/ssp_ble_v1.yaml` as the canonical contract location.
2. Validate and freeze the initial event layout and GATT UUIDs.
3. Implement the generator and templates.
4. Generate C definitions, Kotlin definitions and documentation.
5. Establish shared valid and invalid test vectors.
6. Add deterministic generation and stale-file checks.
7. Run the protocol tests.

Do not manually construct a second set of protocol constants in the firmware or Android application.

## Action 3 — Implement Batch B

Create the separate Zephyr BLE peripheral with synthetic events. Compile in `/tmp/ssp-ble-demo`, fix errors, and flash only after the firmware build passes.

Verify advertising, connection, service discovery, subscription and notification delivery using actual hardware.

## Action 4 — Implement Batch C

Build the minimal Android client, decode notifications, validate events and display the monitoring state.

The UI must visibly show the initial state and the transition to `ELEVATED`.

## Action 5 — Implement Batch D

Automate the software checks, run the complete integration test and preserve the evidence.

Keep software-only and hardware-integration results separate.

## Action 6 — Commit the verified implementation

Commit the protocol, generator, tests, firmware, Android implementation and documentation in coherent changes. Preserve the original Hello World baseline.

---

# 14\. Definition of success for the current phase

The immediate BLE phase is successful when all of the following are demonstrated:

1. The YAML contract is authoritative and internally consistent.
2. Generated C and Kotlin protocol implementations agree with shared test vectors.
3. The Zephyr BLE peripheral builds successfully.
4. The nRF52840 advertises and exposes the expected GATT service.
5. A connected client subscribes to the event characteristic and receives valid notifications.
6. The Android application receives and validates the event.
7. A valid `MOTION_EVENT` causes `NORMAL → ELEVATED`.
8. Invalid packets do not cause the transition.
9. The UI displays the received event and resulting state.
10. The demonstration is repeatable, and the evidence is associated with a known Git revision.

The first version may use synthetic motion events. Physical IMU validation is a subsequent stage, but the architecture must allow the synthetic producer to be replaced cleanly.

---

# 15\. Continuation instruction for ChatGPT

When this document is supplied in a new session, proceed as an engineering collaborator.

Start by determining the actual repository state. Do not repeat the entire roadmap unless requested.

Then implement the next batch, using complete file generation where appropriate and automated verification at the boundaries. Explain precisely which files are being created or changed, which commands should be run, and what evidence is required for the next gate.

Do not assume a source file is correct because it was generated, or assume an acceptance gate passed because a command was proposed.

**The priority is to reach a reliable, repeatable BLE Device → Android Edge demonstration as quickly as possible, while preserving the proven Hello World baseline and avoiding duplicated protocol definitions.**

## Recommended next step

Save the document, then continue from the actual repository state rather than the proposed target structure.

If you want to create it directly from your terminal, copy the document body into `docs/architecture/SSP_PROJECT_BOOTSTRAP.md` and commit it once reviewed.

Your next implementation milestone is Batch A: the protocol contract, generator, templates and shared test vectors. Once that is verified, proceed immediately to the synthetic-event Zephyr BLE peripheral.
