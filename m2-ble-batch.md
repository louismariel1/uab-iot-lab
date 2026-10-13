# Next step: Batch 1 — Define the BLE contract and generate both endpoints

My recommendation is to make the next step a contract-first BLE code-generation batch, not another audit and not a manual implementation of individual source files.

The objective is to establish one authoritative SSP BLE specification and use it to implement the first working communication path across Zephyr and Android.

The first demonstration should be:

`Synthetic MOTION_EVENT` → nRF52840 BLE notification → Android packet validation → `NORMAL → ELEVATED`.

Physical IMU integration can follow without redesigning the communication layer.

## 1. The approach: YAML + templates + automated build

Use three complementary mechanisms:

- YAML DSL: defines the protocol once, including UUIDs, payload layout, event identifiers and versioning.
- Code templates/generator: produces C and Kotlin constants, serialization helpers and shared test vectors.
- Batch orchestrator: generates the code, builds both applications, runs tests and reports exactly which stage failed.

Do not generate the entire firmware or Android application from YAML. Generate the repetitive protocol definitions and use ordinary, maintainable C and Kotlin for the application logic.

1. SSP BLE protocol YAML

Authoritative contract

C generator

UUIDs, encoder, constants

Kotlin generator

UUIDs, decoder, constants

Automated integration batch

Build → unit tests → flash → BLE notifications → Android state

## 2. Proposed repository changes

Build on your existing repository rather than introducing another project structure.

text
```
ssp-iot-poc/
├── device/
│   ├── firmware/
│   │   ├── hello_world/             # Preserve known-good baseline
│   │   └── ble_peripheral/          # New Zephyr application
│   └── protocol/
│       ├── ssp_ble_v1.yaml          # Authoritative protocol
│       └── generated/
│           └── ssp_ble.h            # Generated C definitions
├── edge/
│   ├── protocol/
│   │   └── generated/
│   │       └── SspBleProtocol.kt    # Generated Kotlin definitions
│   ├── tests/
│   │   └── vectors/
│   │       └── ssp_ble_v1.json      # Shared test vectors
│   └── android/                     # Use existing app if available
├── docs/
│   └── protocol/
│       └── ssp-ble-v1.md
└── tools/
    ├── generate_ssp_ble.py
    └── build_ble_demo.sh
```

The Android path is a proposed location, not an assertion that an Android project already exists. If the project is elsewhere, the batch should target that location.

## 3. Define the protocol once

The YAML should specify the following contract:

| Definition | Initial decision |
| --- | --- |
| BLE role | nRF52840 peripheral; Android central |
| GATT service | Custom SSP service |
| Device status | Read characteristic |
| SSP events | Notify characteristic |
| Event types | `NONE`, `MOTION_EVENT` |
| Encoding | Versioned binary payload |
| Byte order | Explicitly defined |
| Measurement time | Device uptime in milliseconds |
| Ordering | Sequence number |
| Test mode | Deterministic synthetic event |
| Edge states | `NORMAL`, `ELEVATED` |

The generator must produce the same UUIDs, event codes, field offsets and encoding rules for both languages. UUIDs and the exact payload layout should be finalized in the protocol file, not independently embedded in C and Kotlin.

## 4. The implementation batch

I would make this a single orchestrated batch with five gates.

1. Gate A — Generate
   Validate YAML, generate C/Kotlin definitions and produce shared test vectors. Fail if generated files are stale.
2. Gate B — Build firmware
   Build the Zephyr peripheral in a dedicated directory such as `/tmp/ssp-ble-demo`. Implement advertising, GATT discovery support, status reads and event notifications.
3. Gate C — Build Android
   Implement BLE scanning, connection management, service discovery, notification subscription, payload validation and the monitoring state machine. Run decoder and state-transition tests.
4. Gate D — Test both endpoints
   Verify valid and malformed payloads against shared test vectors. Test that `NONE` leaves the state unchanged and a valid `MOTION_EVENT` causes `NORMAL → ELEVATED`.
5. Gate E — Hardware demonstration
   Flash the BLE firmware and connect using the Android application. Trigger a synthetic event and confirm the received event and resulting state are visible in the UI.

The orchestrator should stop on the first failure, retain the logs and report a separate result for each gate. It must not automatically commit code or modify your existing Hello World smoke-test script.

## 5. What constitutes success?

The first batch is complete when:

- Both endpoints use the same generated protocol definitions.
- The Zephyr application builds and advertises the SSP service.
- Android discovers the service and subscribes successfully.
- Android receives and validates a real BLE notification from the board.
- A synthetic `MOTION_EVENT` produces the visible `NORMAL → ELEVATED` transition.
- The complete sequence can be repeated, with sequence numbers and timestamps checked.
- The existing Hello World baseline remains intact.

This proves the BLE communication and Edge processing path. It does not yet prove physical motion detection, which remains a separate acceptance requirement.

## 6. Why this is the optimal next step

The biggest risk is not writing BLE APIs; it is letting the firmware and Android implementations drift apart while being developed independently. A shared DSL and generated test vectors address that risk directly.

The batch also provides the earliest useful SSP demonstration without waiting for the physical IMU. Once sensor acquisition is ready, it can feed the existing event interface rather than requiring another BLE implementation.

My recommendation: implement the complete synthetic-event-to-Android BLE vertical slice as the next batch, with the protocol YAML and generator at its center. Do not spend another iteration on audits, and do not make physical IMU integration a prerequisite for starting BLE.
