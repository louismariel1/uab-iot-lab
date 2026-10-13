# SSP BLE implementation: detailed execution plan

The optimal next step is to implement the BLE communication layer through a YAML-first, template-driven, automated build pipeline. We should organize the work into small, verifiable substeps, but execute related substeps in coherent batches rather than manually editing and testing one file at a time.

The first target is a working synthetic-event-to-Android demonstration. Physical IMU integration can then replace the synthetic event producer without redesigning BLE.

## Overview of the implementation batches

Batch 1 — Protocol contract

Steps 1–4 · YAML, UUIDs, payload layout, generation rules and test vectors.

Batch 2 — Zephyr BLE peripheral

Steps 5–7 · GATT service, advertising, notifications and synthetic events.

Batch 3 — Android Edge

Steps 8–10 · BLE lifecycle, decoder, state machine and visible UI.

Batch 4 — Automated integration

Steps 11–13 · Build orchestration, hardware integration and reproducible evidence.

The steps below specify the inputs, actions and acceptance gates for each batch.

## Batch 1 — Establish the protocol contract

### Step 1. Define the BLE protocol in YAML

Create `device/protocol/ssp_ble_v1.yaml`.

It must define:

- Protocol name and version.
- Custom 128-bit service and characteristic UUIDs.
- Device status and event characteristic properties.
- Event identifiers: `NONE` and `MOTION_EVENT`.
- Payload field names, types, widths, offsets and byte order.
- Units and timestamp semantics.
- Device ID and sequence-number representation.
- Valid status values and validation rules.
- Compatibility rules for unsupported versions and malformed messages.

Acceptance gate: the protocol contains no unresolved field sizes, UUID placeholders or ambiguous encoding rules.

### Step 2. Freeze the minimum event payload

Select a compact, versioned binary event format.

For example, the initial contract could include:

| Field | Type | Purpose |
| --- | --- | --- |
| Version | `uint8` | Payload version |
| Event type | `uint8` | `NONE` or `MOTION_EVENT` |
| Status | `uint8` | Device/event validity |
| Flags | `uint8` | Reserved or defined flags |
| Device ID | `uint32` | Device identity |
| Sequence | `uint32` | Event ordering |
| Timestamp | `uint32` | Uptime in milliseconds |
| Motion metric | `uint16` | Quantized movement value |

This candidate occupies 18 bytes. Its final scaling, reserved-bit rules and event semantics must be specified before implementation.

Acceptance gate: firmware and Android can independently derive the exact same byte sequence from the same logical event.

### Step 3. Define the generator templates

Create a small generator, such as `tools/generate_ssp_ble.py`, and templates for:

- C protocol header and serialization helpers.
- Kotlin protocol constants and decoder definitions.
- Protocol reference documentation.
- Shared test-vector fixtures.

The generator should read YAML, validate it, and produce deterministic output. Generated files should contain a warning that manual edits will be overwritten.

Acceptance gate: running the generator twice with unchanged input produces identical output.

### Step 4. Create shared test vectors

Create `device/protocol/vectors/ssp_ble_v1.json`.

Include:

- Valid `NONE` event.
- Valid `MOTION_EVENT`.
- Boundary values for sequence, timestamp and motion metric.
- Incorrect payload length.
- Unsupported protocol version.
- Unknown event type.
- Invalid status or reserved flags.

Both the C and Kotlin implementations must run against the same vectors.

Acceptance gate: both implementations agree on the expected bytes and reject the same invalid cases.

## Batch 2 — Generate the Zephyr BLE peripheral

### Step 5. Create the new firmware application

Create `device/firmware/ble_peripheral/` without modifying `device/firmware/hello_world/`.

Use a structure similar to:

text
```
device/firmware/ble_peripheral/
├── CMakeLists.txt
├── prj.conf
├── Kconfig
└── src/
    ├── main.c
    ├── ble/
    │   ├── ssp_gatt.c
    │   └── ssp_advertising.c
    ├── ssp/
    │   ├── ssp_event.c
    │   └── ssp_protocol.c
    └── test/
        └── synthetic_events.c
```

The optional `Kconfig` file is useful if we want configurable test mode, notification interval and device identity. It is not necessary to introduce it if ordinary configuration options are sufficient.

Responsibilities:

- `main.c`: initialize the application and start the relevant components.
- `ssp_gatt.c`: define the service, characteristics and notification behavior.
- `ssp_advertising.c`: configure advertising data and device name.
- `ssp_protocol.c`: encode logical events according to the generated contract.
- `synthetic_events.c`: produce deterministic test events.

Acceptance gate: the new application has clear module boundaries, and the existing Hello World application remains unchanged.

### Step 6. Implement the GATT service and advertising

Implement the Zephyr peripheral using the Bluetooth APIs available in the actual project workspace.

The device should:

1. Initialize the Bluetooth subsystem.
2. Register the SSP GATT service.
3. Expose a readable device-status characteristic.
4. Expose a notifiable event characteristic.
5. Start advertising with the SSP service UUID.
6. Track connection and disconnection.
7. Enable and disable event notifications according to subscription state.

Use Zephyr's GATT service macros and notification APIs rather than implementing ATT/GATT from scratch.

Keep the service definition tied to the generated protocol constants. If the generator produces a characteristic UUID, the firmware must consume that generated definition rather than hardcoding another copy.

Acceptance gate: a BLE scanner can discover the device, connect, discover the SSP service and inspect its characteristics.

### Step 7. Connect the synthetic event producer

Initially, generate events from a controlled software source.

For example:

text
```
Application starts
       ↓
Publish status
       ↓
Wait for BLE subscription
       ↓
Generate synthetic MOTION_EVENT
       ↓
Encode using generated protocol
       ↓
Send notification
       ↓
Increment sequence
       ↓
Repeat test scenario
```

Make test mode configurable. A development build might emit a `MOTION_EVENT` after a defined delay or on a test trigger, while a later production configuration will use the actual motion detector.

Acceptance gate: a BLE client receives a correctly encoded notification, and the sequence and timestamp fields behave as specified.

## Batch 3 — Generate the Android Edge implementation

### Step 8. Establish the Android application target

The repository tree provided so far does not show an Android application module. This step should first determine whether one already exists in another location in the repository; if not, create a minimal Kotlin Android application module.

Suggested structure:

text
```
edge/android/
├── settings.gradle.kts
├── build.gradle.kts
└── app/
    ├── build.gradle.kts
    └── src/
        ├── main/
        │   ├── AndroidManifest.xml
        │   └── java/.../ssp/
        │       ├── ble/
        │       ├── protocol/
        │       ├── state/
        │       └── ui/
        └── test/
            └── java/.../ssp/
```

Use the project's existing Gradle conventions if an application already exists rather than creating a competing build.

Include the required Android BLE permissions for the selected Android versions, handle runtime permissions, and account for scan/connect restrictions.

Acceptance gate: the Android project builds independently of a physical BLE connection.

### Step 9. Implement the BLE client

Implement the Android BLE lifecycle:

1. Request the required permissions.
2. Scan for the SSP service.
3. Connect to the selected device.
4. Discover GATT services.
5. Verify the expected SSP service and characteristics.
6. Read device status.
7. Enable event notifications using the appropriate characteristic configuration and local callback registration.
8. Receive notification data.
9. Disconnect and reconnect cleanly when required.

Keep the BLE transport separate from payload decoding and SSP state management.

Acceptance gate: Android can connect to the nRF52840, subscribe to the event characteristic and receive notifications.

### Step 10. Implement packet validation, state and UI

Use the generated Kotlin definitions and shared test vectors to decode the payload.

The processing path should be:

text
```
BLE notification
       ↓
Payload length and version validation
       ↓
Event and status validation
       ↓
Sequence/timestamp checks
       ↓
Valid SSP event
       ↓
NORMAL → ELEVATED
       ↓
Update UI and event log
```

The UI should display at least:

- BLE connection state.
- Device ID.
- Latest valid sequence and timestamp.
- Latest received event.
- Current SSP monitoring state.
- Invalid-packet and connection-error indicators.

Avoid treating an invalid packet as a valid motion event. Define how duplicates, out-of-order notifications and reconnects are handled.

Acceptance gate: a valid `MOTION_EVENT` changes the state from `NORMAL` to `ELEVATED`, and the transition is visible to the user.

## Batch 4 — Automate builds, tests and integration

### Step 11. Build the batch orchestrator

Create `tools/build_ble_demo.sh`, with a supporting generator command or Python entry point.

It should execute these stages:

| Stage | Action | Failure behavior |
| --- | --- | --- |
| 1 | Validate YAML and generator inputs | Stop |
| 2 | Generate C/Kotlin definitions and documentation | Stop |
| 3 | Check generated files are current | Stop |
| 4 | Run protocol and encoder/decoder tests | Stop |
| 5 | Build Zephyr firmware | Stop |
| 6 | Build Android application | Stop |
| 7 | Run Android unit tests | Stop |
| 8 | Produce a test summary | Report results |

Use a dedicated build directory, such as `/tmp/ssp-ble-demo`, for the BLE firmware. Preserve `/tmp/ssp-hello-world` for the known-good baseline.

Keep hardware-dependent steps optional so the generation, compilation and unit-test stages can run even when the board is disconnected.

Acceptance gate: one command can regenerate the contract, build both endpoints and run the software tests without manual source-file editing.

### Step 12. Perform the real BLE integration test

Once both builds pass:

1. Run the existing Hello World smoke test if the hardware baseline needs verification.
2. Flash the new BLE application.
3. Start the Android application.
4. Connect and subscribe to the SSP event characteristic.
5. Trigger a synthetic event.
6. Verify the received bytes, decoded fields and sequence number.
7. Confirm `NORMAL → ELEVATED`.
8. Repeat the test, including disconnect/reconnect behavior.

Do not reuse the Hello World UART test unchanged for the BLE firmware. Give the new application its own application-specific test. UART can still report advertising, connection, subscription and event-generation diagnostics.

Acceptance gate: an actual BLE notification from the board produces the expected Android event and state transition.

### Step 13. Commit the reproducible implementation

After the integration test passes, commit:

- The authoritative YAML protocol.
- Generator and templates.
- Generated definitions and test vectors.
- Zephyr BLE application.
- Android implementation.
- Automated tests and build commands.
- Protocol documentation.
- Integration test evidence.

Record the firmware build identity, Git commit, device details and test result. Keep generated artifacts reproducible from their source definitions.

Acceptance gate: another checkout of the repository can regenerate and build the implementation, and the documented hardware procedure reproduces the BLE demonstration.

## 8. Execution plan and priority

I would execute the steps in these four batches:

| Batch | Steps | Deliverable |
| --- | --- | --- |
| A | 1–4 | Frozen protocol contract, generator and shared test vectors |
| B | 5–7 | Working Zephyr BLE peripheral with synthetic events |
| C | 8–10 | Android BLE client, validation and visible SSP state |
| D | 11–13 | One-command builds, integration evidence and reproducible commit |

Within Batch A, freeze the contract before generating endpoint code. B and C can then proceed in parallel against that contract, followed by Batch D.

## 9. The immediate next action

Start with Batch A, Steps 1–4, and produce the protocol YAML, generator templates and shared test vectors together.

Before finalizing the contract, make these decisions explicit:

- The UUIDs and characteristic properties.
- The exact binary payload layout and units.
- The device ID and timestamp representation.
- The handling of invalid packets, duplicates and reconnects.
- The synthetic event trigger used for the first demonstration.

Once those are fixed, generate the protocol artifacts and implement the Zephyr and Android endpoints in parallel. That is the best balance between speed, consistency and testability: it avoids manual cross-language duplication while keeping the generated code small enough to inspect and debug.
