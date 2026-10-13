# Recommended next step: a contract-first BLE implementation batch

I agree with the strategic change. We should stop treating SSP development as a sequence of individual scripts and start implementing a complete, testable Device → Edge vertical slice in coherent batches.

The objective is to get to a live demonstration quickly: an nRF52840 sends an SSP event over BLE, an Android application receives it, validates it, and changes its monitoring state from `NORMAL` to `ELEVATED`.

There is one important engineering distinction: we can implement and demonstrate BLE communication using a deterministic synthetic event source before the physical IMU is ready. That proves the communication architecture and enables parallel Android development. We must still validate the physical sensor before claiming the full physical-movement acceptance test passes.

## 1. Change the implementation strategy

Instead of completing every M2 subtask before starting BLE, use three parallel implementation tracks with one shared protocol contract.

### Track A — Shared contract

Define once; generate or implement both endpoints against it.

- GATT service and characteristic UUIDs
- Versioned event payload and field semantics
- Shared test vectors and validation rules

### Track B — Device

- Zephyr BLE peripheral
- Advertising and GATT
- Notifications
- Synthetic event producer
  nRF52840

### Track C — Android Edge

- Scan and connect
- Discover GATT
- Subscribe to notifications
- Validate and update state
  Kotlin

Integration gate

Synthetic `MOTION_EVENT` → BLE notification → validated Android event → `ELEVATED`

The key optimization is that the BLE protocol becomes the shared contract, not two separately developed implementations that we try to reconcile later.

## 2. Use a small protocol DSL as the source of truth

Rather than manually synchronizing C structs, Kotlin data classes, UUIDs and test payloads, define the protocol once in a machine-readable file.

Recommended location:

text
```
device/protocol/
├── ssp_ble_v1.yaml          # Authoritative contract
├── schema/
│   └── ssp_ble.schema.json  # Optional schema validation
├── vectors/
│   └── ssp_ble_v1.json      # Shared test vectors
└── README.md
```

The YAML file describes the protocol; a small generator creates the language-specific constants and test fixtures. The firmware and Android application retain their own native implementations, but they derive their definitions from the same contract.

For example, the DSL could specify:

yaml
```
protocol:
  name: SSP BLE
  version: 1

device:
  role: peripheral
  platform: nrf52840
  rtos: zephyr

service:
  name: SSP Service
  uuid: REQUIRED_CUSTOM_UUID

characteristics:
  - name: device_status
    properties: [read]
  - name: event_data
    properties: [read, notify]

events:
  NONE: 0
  MOTION_EVENT: 1

edge_states:
  NORMAL:
    on: MOTION_EVENT
    next: ELEVATED

  ELEVATED:
    on: MOTION_EVENT
    next: ELEVATED

payload:
  encoding: binary
  byte_order: little_endian
  versioned: true
```

This is a template illustrating the approach, not a finalized protocol specification. The UUIDs, exact payload layout, field units, permissions and state-transition behavior must be agreed upon before code generation.

### What should be generated?

- C header files for protocol constants, event identifiers and payload encoding.
- Kotlin constants, event definitions and decoding support.
- Shared binary test vectors for valid, malformed and unsupported-version packets.
- Protocol documentation and a traceability table linking requirements to tests.

Do not try to generate the entire application from YAML. Generate the repetitive, cross-language contract and keep the Bluetooth lifecycle and business logic in maintainable native code.

## 3. Freeze a minimal BLE contract before generating the batch

For the first demonstration, I recommend two GATT characteristics.

| Characteristic | Purpose |
| --- | --- |
| Device status | Read device identity, firmware/protocol version and operational state |
| Event data | Notify the Edge application of validated SSP events |

Use a custom 128-bit service UUID and characteristic UUIDs. The protocol must define exact byte order, event codes, field widths and version handling.

For the event payload, keep the first version compact. One reasonable candidate is an 18-byte fixed-length record containing:

| Field | Size |
| --- | --- |
| Protocol version | 1 byte |
| Event type | 1 byte |
| Status | 1 byte |
| Reserved/flags | 1 byte |
| Device ID | 4 bytes |
| Sequence number | 4 bytes |
| Device uptime timestamp | 4 bytes |
| Motion metric, quantized | 2 bytes |
| Total | 18 bytes |

This is a proposed layout, not a committed specification. It fits within the 20-byte notification value available with the default ATT MTU of 23 bytes. Keeping raw acceleration out of the initial event packet also aligns with the PoC's data-minimization principle.

The decoder must reject incorrect lengths, unsupported versions and invalid event codes. Sequence and timestamp validation should account for reconnects, device restarts and counter wraparound.

Zephyr provides GATT service and notification APIs for this design, including `bt_gatt_notify()`. Its documented peripheral sample is a useful implementation reference. docs.zephyrproject.org

+2

 Android's BLE architecture likewise supports the required scan, connect, service-discovery and data-transfer sequence. Android Developers

+1

## 4. Implement the largest coherent batch

I would use a single implementation batch with automated generation, build and test stages—not one script per source file.

Batch 1 — Contract and fixtures

Define YAML, UUIDs, payload layout, shared test vectors and protocol documentation.

Batch 2 — Zephyr BLE peripheral

Add advertising, GATT service, status characteristic, notification characteristic and a deterministic synthetic event producer.

Batch 3 — Android Edge

Implement scanning, connection management, GATT discovery, notification subscription, packet parsing and the NORMAL/ELEVATED state machine.

Batch 4 — Automated verification

Generate both endpoints, build firmware and Android, run unit tests, flash the device and verify actual BLE communication.

A batch orchestrator should perform these operations in order:

1. Validate the protocol definition.
2. Generate protocol constants and fixtures.
3. Build the Zephyr application in a dedicated directory.
4. Run firmware unit tests and Android parser/state-machine tests.
5. Build the Android application.
6. Flash the BLE firmware when hardware is available.
7. Execute the BLE integration test and save logs and results.
8. Report each gate separately and stop at the first failed prerequisite.

This is much more efficient than manually editing files and testing each one. However, it should not automatically overwrite the known-good Hello World application or silently change the protocol when a test fails.

## 5. Keep the physical sensor independent from the BLE work

The first demonstrable system should use a synthetic event source:

Synthetic motion producer

SSP event model and encoder

Zephyr BLE GATT notifications

Android decoder and validation

NORMAL → ELEVATED

Use a deterministic trigger, such as a periodic test event or a development-only command, to exercise the complete path. Label these events as synthetic so they cannot be mistaken for physical sensor detections.

Once the IMU is integrated, replace the event producer with the actual acquisition and detection pipeline. The encoder, GATT service, Android parser and state machine should remain unchanged.

## 6. One prerequisite before generating code

Your previous build output identifies the Zephyr source as `v4.5.0-rc1-43-ga90cd40a5f0a`. That is a prerelease build identity, even though your Python environment is named `zephyr-3.12`. We should confirm the active Zephyr revision and inspect the actual BLE samples before generating firmware, to avoid targeting APIs or configuration from the wrong release.

Also, the repository tree currently shows `edge/protocol` and `edge/tests`, but no visible Android application module. The generator needs to know where the Kotlin application actually lives—or we need to establish that module as part of the batch.

## 7. My recommended immediate next action

Run a short repository/toolchain audit, then implement the contract-first BLE batch. Do not spend the next iteration manually creating the synthetic M2 files one at a time.

The audit can be performed in one terminal batch:

bash
```
cd ~/ssp-iot-poc

echo "=== Git baseline ==="
git status --short
git log -1 --oneline

echo "=== Zephyr revision ==="
cd ~/zephyrproject
west topdir
west list zephyr
git -C zephyr describe --tags --always --dirty

echo "=== Available BLE samples ==="
find zephyr/samples/bluetooth \
  -maxdepth 2 \
  -name CMakeLists.txt \
  -print | sort

echo "=== SSP repository structure ==="
cd ~/ssp-iot-poc
find device edge docs tools -maxdepth 4 -type f -print | sort
```

The outputs will establish the actual source revision, available BLE reference applications, current Git baseline and location of the Android project.

From that audit, the next deliverable should be a single reviewable implementation batch containing the protocol DSL, generated contract files, Zephyr BLE peripheral, Android BLE client, automated tests and dedicated build commands.

Keep M2's physical-sensor acceptance criteria explicit, but allow the BLE and Edge implementation to progress in parallel. That is the fastest route to a meaningful SSP demonstration without confusing a successful BLE prototype with a completed physical-sensing milestone.
