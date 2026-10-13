After implementing Batches A–D, the target repository should preserve your existing SSP structure while adding a protocol-generation pipeline, a Zephyr BLE peripheral, an Android BLE client, automated tests, and reproducible integration evidence.

The key principle is to keep the existing Hello World baseline intact and add BLE as a separate firmware application.

## 1. Target repository structure

text
```
ssp-iot-poc/
├── README.md
├── .gitignore
│
├── protocol/
│   └── ble/
│       ├── ssp_ble_v1.yaml
│       ├── README.md
│       └── test_vectors.json
│
├── tools/
│   ├── compile_and_flash_test.sh          # Existing
│   ├── flash_and_uart_smoke_test.sh       # Existing
│   ├── build_ble_demo.sh                  # Batch D
│   └── ble_codegen/
│       ├── generate.py
│       ├── templates/
│       │   ├── protocol_constants.h.j2
│       │   ├── protocol_codec.h.j2
│       │   ├── ProtocolConstants.kt.j2
│       │   └── ProtocolCodec.kt.j2
│       └── requirements.txt
│
├── device/
│   ├── drivers/
│   │
│   ├── protocol/
│   │   └── generated/
│   │       ├── ssp_ble_constants.h
│   │       └── ssp_ble_codec.h
│   │
│   ├── firmware/
│   │   ├── hello_world/                   # Existing, unchanged
│   │   │   ├── CMakeLists.txt
│   │   │   ├── prj.conf
│   │   │   └── src/
│   │   │       └── main.c
│   │   │
│   │   └── ble_peripheral/                # Batch B
│   │       ├── CMakeLists.txt
│   │       ├── prj.conf
│   │       ├── boards/
│   │       │   └── nrf52840dk_nrf52840.conf
│   │       └── src/
│   │           ├── main.c
│   │           ├── ble/
│   │           │   ├── ssp_ble_service.c
│   │           │   ├── ssp_ble_service.h
│   │           │   ├── ssp_ble_advertising.c
│   │           │   └── ssp_ble_advertising.h
│   │           ├── sensor/
│   │           │   ├── sensor_interface.h
│   │           │   └── sensor_sim.c
│   │           └── ssp/
│   │               ├── ssp_types.h
│   │               ├── ssp_motion.c
│   │               └── ssp_motion.h
│   │
│   ├── tests/
│   │   ├── unit/
│   │   │   ├── test_motion.c
│   │   │   └── test_codec.c
│   │   └── integration/
│   │       └── README.md
│   │
│   └── CMakeLists.txt                     # Optional shared configuration
│
├── edge/
│   ├── android/                           # Batch C
│   │   ├── settings.gradle.kts
│   │   ├── build.gradle.kts
│   │   ├── gradlew
│   │   ├── gradle/
│   │   └── app/
│   │       ├── build.gradle.kts
│   │       └── src/
│   │           ├── main/
│   │           │   ├── AndroidManifest.xml
│   │           │   └── java/
│   │           │       └── .../
│   │           │           ├── ble/
│   │           │           │   ├── SspBleClient.kt
│   │           │           │   ├── SspGatt.kt
│   │           │           │   └── BleConnectionState.kt
│   │           │           ├── protocol/
│   │           │           │   ├── ProtocolConstants.kt
│   │           │           │   └── ProtocolCodec.kt
│   │           │           ├── model/
│   │           │           │   ├── SspEvent.kt
│   │           │           │   └── SspState.kt
│   │           │           ├── state/
│   │           │           │   └── SspStateReducer.kt
│   │           │           └── ui/
│   │           │               └── .../
│   │           └── test/
│   │               └── java/
│   │                   └── .../
│   │                       ├── ProtocolCodecTest.kt
│   │                       └── SspStateReducerTest.kt
│   │
│   ├── protocol/
│   │   └── README.md
│   └── tests/
│       └── test_vectors/
│           └── README.md
│
├── cloud/                                 # Deferred
│   ├── backend/
│   ├── database/
│   └── frontend/
│
├── ml/                                    # Deferred
│
└── docs/
    ├── architecture/
    │   └── device-edge-architecture.md
    ├── protocol/
    │   └── ssp-ble-protocol-v1.md
    ├── testing/
    │   ├── pcb-flash-smoke-test.md
    │   └── ble-integration-test.md
    └── reports/
        └── ble-demo/
            └── README.md
```

The `.../` package directories under Android should be replaced with your actual Kotlin package name. The exact division of Kotlin classes can be adjusted when integrating with the UI framework you select.

## 2. What each batch creates

Batch A — Protocol and generator

Foundation

- `protocol/ble/ssp_ble_v1.yaml` is the protocol source of truth.
- `tools/ble_codegen/` generates C and Kotlin artifacts.
- `test_vectors.json` supplies shared cross-language test cases.
- `docs/protocol/ssp-ble-protocol-v1.md` documents the contract.

Batch B — Zephyr BLE peripheral

- `device/firmware/ble_peripheral/` contains the separate Zephyr application.
- Synthetic events exercise the BLE service before IMU integration.
- GATT characteristics expose device information and event notifications.
- Unit tests cover encoding and motion-event generation.

Batch C — Android Edge

- `edge/android/` is the Android Gradle project.
- BLE scanning, GATT discovery and notifications are implemented in `ble/`.
- `protocol/` validates incoming packets.
- `state/` transitions the monitoring state from `NORMAL` to `ELEVATED`.
- `ui/` displays the connection, events and current state.

Batch D — Automated integration

- `tools/build_ble_demo.sh` orchestrates generation, tests and builds.
- `docs/testing/ble-integration-test.md` defines the hardware procedure.
- `docs/reports/ble-demo/` records reproducible results.
- Git captures the protocol, implementation and test evidence.

## 3. Important repository design decisions

### A. Keep Hello World separate

Your existing scripts use `/tmp/ssp-hello-world`. Preserve that build directory and application as the known-good hardware diagnostic.

Use separate build directories:

| Application | Build directory |
| --- | --- |
| Hello World baseline | `/tmp/ssp-hello-world` |
| BLE peripheral | `/tmp/ssp-ble-demo` |

This prevents a BLE build from overwriting the baseline firmware artifacts.

### B. Keep generated code separate from templates

The source of truth should be:

`protocol/ble/ssp_ble_v1.yaml`

The generator and templates live in `tools/ble_codegen/`. Generated C and Kotlin files live with their respective consumers.

Do not manually edit generated files. If the protocol changes, update YAML, regenerate both language implementations, and rerun the shared tests.

### C. Use one shared test-vector file

`protocol/ble/test_vectors.json` should contain valid and invalid packets, including:

- Valid `NONE` and `MOTION_EVENT` packets.
- Sequence and timestamp examples.
- Invalid packet lengths.
- Unsupported protocol versions.
- Invalid event types and status values.

The C encoder and Kotlin decoder should both be tested against these vectors. This is more reliable than separately implementing the same assumptions in two languages.

### D. Defer cloud implementation

Keep the existing `cloud/` and `ml/` directories unchanged for now. The first demonstration should establish:

Synthetic or physical movement → nRF52840 → BLE notification → Android validation → `ELEVATED`.

Once this path is stable, the same architecture can be extended to the physical IMU, adaptive monitoring and cloud integration.

## 4. What should be committed to Git?

Commit source files, protocol definitions, templates, generated source, tests and documentation. Do not commit temporary build output, compiled firmware binaries, local SDK installations, Python virtual environments, or machine-specific absolute paths.

The final Batch D report should identify the Git commit, Zephyr revision, board target, firmware build result, Android test result, and actual BLE integration outcome.

Recommended next action: implement Batch A and Batch B first, with the BLE peripheral generating deterministic synthetic motion events. Then implement the Android receiver against exactly the same YAML-defined protocol. You do not need to complete the IMU, cloud or ML layers before demonstrating the core SSP Device → Edge architecture.
