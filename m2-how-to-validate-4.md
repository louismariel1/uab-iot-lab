# SSP BLE generated-code verification plan

Your latest output is sufficient to establish the initial verification baseline, but it is not sufficient to verify every generated module or determine which code is correct.

The key finding is that `tools/verification.sh` reports no BLE-related files and lists only the three scripts in `tools/`. This suggests that the BLE implementation described in the previous plan has not yet been generated in the repository. We should confirm that using a more complete inventory before creating or modifying anything.

## 1. Current verification status

Based on the terminal output you provided:

Existing Hello World firmware

Previously verified

`device/firmware/hello_world/` and the two existing build/flash scripts are present. Your earlier test output confirms successful compilation, flashing, and UART smoke testing.

BLE implementation files

Not verified

No BLE-related files were reported by `verification.sh`. The current tree contains no visible BLE peripheral, Android app, YAML contract, or generator.

`tools/verification.sh`

Not verified

The file exists but is untracked (`?? tools/verification.sh`). Its output does not establish whether the script checks the entire repository or only selected directories and markers.

## 2. Step 1 — Complete the actual file inventory

Run this single command block from `~/ssp-iot-poc`. It does not modify the repository.

bash
```

cd ~/ssp-iot-poc

echo "=== 1. Git baseline ==="
git status --short
git log -1 --oneline

echo
echo "=== 2. All tracked and untracked files ==="
git ls-files
find . -path './.git' -prune -o -type f -print | sort

echo
echo "=== 3. BLE/protocol/generator candidates ==="
find device edge tools tests protocol docs \
  -type f \
  \( -iname '*ble*' \
     -o -iname '*ssp*protocol*' \
     -o -iname '*generate*' \
     -o -iname '*codegen*' \
     -o -iname '*vector*' \
     -o -iname '*android*' \
     -o -iname '*gradle*' \
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

echo
echo "=== 4. Verification script ==="
sed -n '1,240p' tools/verification.sh

echo
echo "=== 5. Generated-code markers across repository ==="
grep -RInE \
  --exclude-dir=.git \
  --exclude='verification.sh' \
  'SSP_PROTOCOL_VERSION|SSPEventV1|synthetic_motion_event|BT_GATT_SERVICE_DEFINE|class SspState|SspProtocol|MOTION_EVENT' \
  device edge tools docs tests protocol 2>/dev/null || true
```

Why this is necessary:

- `ls tools` only inventories the immediate contents of `tools/`.
- `tree` inventories directories and files, but not their correctness or generation history.
- `git status --short` identifies untracked and modified files, not all generated files.
- `verification.sh` may search only a subset of directories or specific strings.

The combined inventory will tell us what exists, what is tracked, and whether any generated code is hiding in a different location.

## 3. Step 2 — Master list of modules and scripts to verify

The following is the provisional verification register. It includes the expected BLE implementation and the existing scripts. The BLE entries are expected deliverables, not claims that the files already exist.

| Order | Module or script | Expected location | Status |
| --- | --- | --- | --- |
| 1 | Verification/inventory script | `tools/verification.sh` | Not verified |
| 2 | Authoritative BLE YAML | `protocol/ssp_ble_v1.yaml` or `device/protocol/ssp_ble_v1.yaml` | Not verified |
| 3 | Generator dependencies | `tools/ble_codegen/requirements.txt` | Not verified |
| 4 | Generator entry point | `tools/ble_codegen/generate.py` | Not verified |
| 5 | C protocol template | `tools/ble_codegen/templates/ssp_protocol.h.j2` | Not verified |
| 6 | Kotlin protocol template | `tools/ble_codegen/templates/SspProtocol.kt.j2` | Not verified |
| 7 | Documentation template | `tools/ble_codegen/templates/protocol.md.j2` | Not verified |
| 8 | Shared protocol test vectors | `tests/ble/test_vectors.json` | Not verified |
| 9 | Protocol unit tests | `tests/ble/test_protocol.py` | Not verified |
| 10 | Generated C protocol header | `device/firmware/ble_peripheral/src/generated/ssp_protocol.h` | Not verified |
| 11 | Zephyr BLE application configuration | `device/firmware/ble_peripheral/CMakeLists.txt`, `prj.conf` | Not verified |
| 12 | Zephyr GATT service | `device/firmware/ble_peripheral/src/ssp_gatt.c` | Not verified |
| 13 | Advertising/connection handling | `device/firmware/ble_peripheral/src/ssp_advertising.c` or equivalent | Not verified |
| 14 | Synthetic event producer | `device/firmware/ble_peripheral/src/ssp_event.c` | Not verified |
| 15 | Firmware entry point | `device/firmware/ble_peripheral/src/main.c` | Not verified |
| 16 | Generated Kotlin protocol | `edge/android/app/src/main/java/org/ssp/protocol/SspProtocol.kt` | Not verified |
| 17 | Android Gradle project | `edge/android/` | Not verified |
| 18 | Android BLE transport/client | `edge/android/app/src/main/java/org/ssp/ble/` | Not verified |
| 19 | Android state machine | `edge/android/app/src/main/java/org/ssp/state/SspState.kt` | Not verified |
| 20 | Android UI and permissions | Android app source and manifest | Not verified |
| 21 | BLE build orchestrator | `tools/build_ble_demo.sh` | Not verified |
| 22 | Existing compile/flash test | `tools/compile_and_flash_test.sh` | Previously passed; rerun if baseline changes |
| 23 | Existing UART smoke test | `tools/flash_and_uart_smoke_test.sh` | Previously passed; rerun if baseline changes |
| 24 | Generated protocol documentation | `docs/protocol/ssp-ble-v1.md` | Not verified |
| 25 | Hardware integration evidence | `docs/testing/` or another agreed location | Not verified |

Important: Some planned files may be consolidated into fewer modules. We should verify the actual architecture rather than create every filename in this table just to match it.

## 4. Step 3 — Reorder verification by dependencies

We should work through the list in this order, marking each item `PASS`, `FAIL`, `NOT PRESENT`, or `BLOCKED`.

1. Repository inventory and baseline
   Confirm the actual files, Git state, and existing verification script. Preserve the known-good Hello World application and scripts.
2. Protocol YAML
   Verify field sizes, offsets, total payload length, UUID uniqueness, event values, device ID, byte order, and the expected payload bytes.
3. Generator and templates
   Check schema validation, deterministic output, invalid-input handling, and whether generated UUIDs and field layouts genuinely come from YAML.
4. Shared test vectors and protocol tests
   Verify valid and invalid packets, including malformed length, unsupported version, unknown event, invalid status, and reserved flags if defined.
5. Generated C and Kotlin protocol implementations
   Verify byte-for-byte agreement with the vectors. Confirm that both implementations reject the same invalid packets.
6. Zephyr BLE peripheral
   Check configuration, compilation, GATT attribute definitions, status reads, advertising, subscription handling, notification delivery, and synthetic-event timing.
7. Android application
   Check Gradle configuration and build first, then permissions, scanning, GATT discovery, CCCD subscription, decoding, sequence handling, state transitions, and UI.
8. Automated build pipeline
   Confirm generation checks, protocol tests, firmware compilation, Android tests, meaningful failure handling, and accurate reporting of skipped stages.
9. Hardware integration
   Flash only after firmware verification. Confirm actual BLE notification bytes, Android decoding, `NORMAL → ELEVATED`, and reconnect behavior.
10. Reproducibility and evidence
    Verify clean regeneration, record the Git revision and build identity, and preserve integration results.

## 5. Step 4 — Establish strict verification rules

For each file, we will record:

- Existence: Is the file actually present?
- Provenance: Was it generated, handwritten, or modified afterward?
- Static correctness: Does its content agree with the protocol and architecture?
- Automated verification: Which tests or build commands pass?
- Integration verification: Has its behavior been demonstrated on hardware, where applicable?

A file's existence is not proof of correctness. Likewise, a successful compilation does not prove BLE interoperability.

We should also avoid treating `--check` as a reliable freshness test until we inspect the generator. In the supplied example, that option renders outputs in memory but does not compare them with existing generated files. A proper freshness check must compare the expected output against the checked-in or working-tree artifact.

## Your immediate next task

Run Step 1's inventory command block and paste its output.

We will then produce the final, repository-specific verification register, removing nonexistent files from the existing-file list while keeping missing items explicitly marked `NOT PRESENT`. We will begin with the first unresolved item and proceed sequentially, without generating or changing code until the inventory is established.
