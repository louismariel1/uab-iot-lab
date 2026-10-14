# SSP IoT PoC — M2 Project Bootstrap and Handover

Project handover • 14 October 2026 • Repository checkpoint: `43efee7`

This bootstrap is intended to be copied into a new ChatGPT conversation so troubleshooting can resume without repeating earlier work, losing important context, or making speculative changes to working code.

## 1. Project overview

Project: SSP IoT Proof of Concept (PoC)

Repository: `~/ssp-iot-poc`

Primary goal: Build an end-to-end IoT proof of concept in which a BLE peripheral exposes SSP protocol data, an Android application connects to it and consumes the data, and protocol encoding is consistent across firmware and Android.

The current milestone, M2, covers the BLE peripheral, shared protocol implementation, Android BLE integration, and associated testing and documentation.

The intended end-to-end flow is:

nRF52840 BLE peripheral

Zephyr firmware, advertising, GATT, event notifications

Bluetooth Low Energy

Connection, GATT characteristics and notifications

Android SSP client

Scan, connect, subscribe, decode and expose state

Verified integration

Protocol tests, integration tests, documented results and hardware validation

The protocol is intended to be the common contract between the firmware and Android. The BLE implementation must use the UUIDs, payload layout, byte order and event definitions specified by that contract.

## 2. Current repository checkpoint

The most recent confirmed Git output was:

text
```
43efee7 (HEAD -> main) Implement SSP BLE peripheral and Android integration
```

The command `git status --short` produced no output before the Git log line. This indicates that the working tree was clean at that checkpoint.

The commit includes staged and committed changes to firmware, protocol definitions, generated code, Android integration, tests, scripts and documentation.

Important: the commit exists locally. No evidence has yet been provided that it was pushed to a remote repository.

## 3. What has been achieved

### 3.1 Zephyr build environment

The firmware build environment is operational.

| Component | Confirmed configuration |
| --- | --- |
| Zephyr workspace | `~/zephyrproject` |
| Application repository | `~/ssp-iot-poc` |
| Zephyr version | `4.5.0-rc1` |
| Build board | `nrf52840dk/nrf52840` |
| Python environment | `zephyr-3.12` |
| Build directory | `/tmp/ssp-ble-peripheral` |

The build command that succeeded was:

bash
```
cd ~/zephyrproject && west build -p always -b nrf52840dk/nrf52840 -d /tmp/ssp-ble-peripheral ~/ssp-iot-poc/device/firmware/ble_peripheral
```

The successful build reached:

text
```
[239/239] Linking C executable zephyr/zephyr.elf
```

The reported memory usage was:

- Flash: 103,448 bytes out of 1 MiB, approximately 9.87%.
- RAM: 22,104 bytes out of 256 KiB, approximately 8.43%.

The build generated both required firmware artifacts:

- `/tmp/ssp-ble-peripheral/zephyr/zephyr.hex`
- `/tmp/ssp-ble-peripheral/zephyr/zephyr.elf`

Both files were checked and found to be non-empty.

### 3.2 BLE advertising

The file `device/firmware/ble_peripheral/src/ssp_advertising.c` was updated to address compatibility issues with the installed Zephyr version.

The current implementation includes:

- Parsing the SSP service UUID from `SSP_SERVICE_UUID_STR`.
- Converting the canonical UUID string to the byte order used in BLE advertising.
- Advertising the general-discoverable and no-BR/EDR flags.
- Advertising the 128-bit SSP service UUID.
- Including the complete device name.
- Using `BT_LE_ADV_CONN_FAST_1` instead of the unavailable `BT_LE_ADV_CONN` macro.

The successful firmware build confirms that this source compiles and links in the current environment.

It does not yet confirm that a phone can discover the peripheral over the air.

### 3.3 GATT implementation

The file `device/firmware/ble_peripheral/src/ssp_gatt.c` was repaired after several UUID macro compatibility and malformed-source issues.

The current implementation uses explicit `BT_UUID_TYPE_128` initializers for the three UUID structures rather than relying on the unavailable `BT_UUID_128_INIT` macro.

The three UUIDs retain these leading byte sequences:

- Service UUID: `3e e9 cf 6e ...`
- Status characteristic UUID: `40 e9 cf 6e ...`
- Event characteristic UUID: `41 e9 cf 6e ...`

The GATT implementation also includes:

- Status characteristic handling.
- Event characteristic notification support.
- Notification subscription tracking.
- Active connection reference management.
- An `-ENOTCONN` guard when notifications are requested without an active connection.

The notification call currently uses `&ssp_service.attrs[4]` as the event characteristic value attribute. This index was reasoned to match the intended GATT layout but still needs runtime validation against the actual registered attribute layout.

### 3.4 Event publishing

The file `device/firmware/ble_peripheral/src/ssp_event.c` previously implemented only a private event-publishing helper. The public lifecycle functions declared by `ssp_event.h` were missing, which caused linker errors for `ssp_event_init` and `ssp_event_start`.

The file was then replaced with a complete implementation containing:

- `ssp_event_init()`
- `ssp_event_start()`
- `ssp_event_subscribed()`
- A Zephyr timer.
- A one-second publishing interval.
- SSP event encoding using the generated protocol code.
- Notification attempts only while GATT notifications are subscribed.
- Error logging if a notification attempt fails.

The motion value is currently synthetic and fixed at `1800 mg`. No physical motion sensor integration has been confirmed.

### 3.5 Shared protocol and code generation

The repository includes the following protocol-related components:

- `protocol/ssp_ble_v1.yaml`
- `tools/ble_codegen/generate.py`
- `tools/ble_codegen/requirements.txt`
- `tools/ble_codegen/templates/ssp_protocol.h.j2`
- `tools/ble_codegen/templates/SspProtocol.kt.j2`
- `tools/ble_codegen/templates/protocol.md.j2`
- `tests/ble/test_protocol.py`
- `tests/ble/test_vectors.json`
- `docs/protocol/ssp-ble-v1.md`

The generated C and Kotlin protocol representations are intended to keep the firmware and Android implementations aligned.

These files were included in the committed changes. However, a successful firmware build alone does not establish that the generator runs cleanly, that all test vectors pass, or that Android decodes the same vectors correctly. Those checks remain part of the validation plan.

### 3.6 Android implementation

The repository contains an Android project structure with the following files:

- `edge/android/settings.gradle.kts`
- `edge/android/build.gradle.kts`
- `edge/android/app/build.gradle.kts`
- `edge/android/app/src/main/AndroidManifest.xml`
- `edge/android/app/src/main/java/org/ssp/ble/AndroidSspBleClient.kt`
- `edge/android/app/src/main/java/org/ssp/ble/SspBleClient.kt`
- `edge/android/app/src/main/java/org/ssp/protocol/SspProtocol.kt`
- `edge/android/app/src/main/java/org/ssp/state/SspState.kt`
- `edge/android/app/src/test/java/org/ssp/protocol/SspProtocolTest.kt`
- `edge/android/app/src/test/java/org/ssp/state/SspStateTest.kt`

The Android BLE client and state-management implementation have been added to the project.

Their presence does not establish that the Android app builds, passes unit tests, or connects successfully to the actual firmware. Those results need to be collected.

### 3.7 Build and test scripts

The repository contains these helper scripts:

- `tools/build_ble_demo.sh`
- `tools/test_ble_integration.sh`
- `tools/list_empty_files.sh`
- `tools/collect_implementation_context.sh`
- `tools/create_android_edge_structure.sh.`
- `tools/create_ble_integration_structure.sh`
- `tools/create_zephyr_ble_structure.sh`

The script `tools/create_android_edge_structure.sh.` has a trailing period in its filename. Verify whether this is intentional before relying on it or renaming it.

The scripts were committed, but their complete behavior and success status have not been established in the conversation.

## 4. Current step: validate the committed checkpoint

The last known successful firmware build predates the final commit. The next step is therefore to verify the repository checkpoint without making any new source changes.

### Step 1 — Confirm the Git state

Run:

bash
```
cd ~/ssp-iot-poc && git status --short && git log -1 --oneline
```

Expected result: no output from `git status --short`, followed by the latest commit.

### Step 2 — Verify the committed diff

Run:

bash
```
cd ~/ssp-iot-poc && git show --stat --oneline HEAD && git diff HEAD^ HEAD --check
```

This checks the committed change summary and whitespace issues in the commit. If `HEAD^` is unavailable, stop and inspect the repository history rather than assuming the comparison succeeded.

### Step 3 — Verify the zero-length files

The three files that were originally identified as empty were:

1. `docs/testing/ssp-ble-test-results.md`
2. `docs/testing/ssp-ble-integration.md`
3. `tools/test_ble_integration.sh`

They appeared as new files in the staged changes and were included in commit `43efee7`. Their current contents have not yet been independently inspected in the latest checkpoint.

Run this single-line command:

bash
```
cd ~/ssp-iot-poc && wc -lc docs/testing/ssp-ble-test-results.md docs/testing/ssp-ble-integration.md tools/test_ble_integration.sh
```

All three files should contain actual content. Nonzero line counts are expected, but the content also needs to be reviewed for completeness.

### Step 4 — Check that the firmware still builds

Use an incremental build first, preserving the existing build directory:

bash
```
cd ~/zephyrproject && west build -b nrf52840dk/nrf52840 -d /tmp/ssp-ble-peripheral ~/ssp-iot-poc/device/firmware/ble_peripheral
```

If the build succeeds, verify both artifacts:

bash
```
test -s /tmp/ssp-ble-peripheral/zephyr/zephyr.hex && test -s /tmp/ssp-ble-peripheral/zephyr/zephyr.elf && echo "PASS: Firmware artifacts verified"
```

Do not edit firmware code just because an unrelated Android or test check fails.

## 5. Remaining zero-length files and their required implementation

The three files above are important M2 deliverables because they document and automate validation. The goal is not merely to make them non-empty; each must contain a meaningful, executable or auditable implementation.

### 5.1 `docs/testing/ssp-ble-test-results.md`

Purpose: provide the M2 verification record.

It should include:

- Firmware build command and result.
- Board target and Zephyr version.
- Firmware artifact paths and sizes.
- Protocol generator result.
- Python protocol test results.
- Android unit test results.
- BLE integration test results.
- Hardware test results, if a physical board is available.
- Known failures, warnings and outstanding limitations.
- Date, Git commit and test environment.

Results must reflect tests that actually ran. Do not write `PASS` for a test that has not been executed.

### 5.2 `docs/testing/ssp-ble-integration.md`

Purpose: describe the repeatable end-to-end BLE integration procedure.

It should cover:

- Prerequisites and environment setup.
- Firmware build and flashing instructions.
- BLE advertising and service discovery.
- Service, status and event characteristic UUIDs.
- Connection and notification subscription.
- SSP payload decoding and protocol version checks.
- Android app execution and required permissions.
- Expected behavior and diagnostic commands.
- Failure recovery and troubleshooting.
- Explicit distinction between simulator/build tests and real hardware tests.

The UUIDs and payload structure must be taken from the current protocol definition and firmware source rather than reconstructed from memory.

### 5.3 `tools/test_ble_integration.sh`

Purpose: automate the integration checks that can run from the development environment.

The script should:

- Fail immediately on errors.
- Resolve the repository root robustly.
- Check required tools and prerequisites.
- Run the protocol tests.
- Run code generation validation.
- Run the Android unit tests if the required Java/Gradle environment is available.
- Verify firmware artifacts or invoke the firmware build as appropriate.
- Return a nonzero exit code when a required check fails.
- Clearly identify tests that require a physical BLE peripheral or Android device.

A host-side script must not claim to have verified radio communication if it only builds the firmware or executes unit tests.

Recommended implementation order: inspect all three files first. If any are already complete, retain them and test them rather than replacing them. Implement only the missing content.

## 6. Next steps after checkpoint validation

The following order minimizes regression risk.

1. Verify the committed checkpoint
   Confirm the clean Git state, inspect the commit summary, validate the three testing files, and rerun the firmware build.
2. Validate protocol generation
   Inspect the generator requirements, run the generator in its documented environment, and verify that generated C, Kotlin and protocol documentation match their source templates and YAML definition.
3. Run protocol unit tests
   Execute `tests/ble/test_protocol.py` and confirm that valid vectors decode correctly, invalid inputs are rejected, and byte order and field sizes agree with the protocol definition.
4. Build and test Android
   Run the Gradle unit tests and Android build. Investigate dependency, SDK, manifest, BLE permission and API compatibility problems independently of the firmware.
5. Implement and run the integration script
   Complete `tools/test_ble_integration.sh`, then run it from the documented environment. Make its output distinguish successful automated checks from unavailable hardware checks.
6. Validate the BLE protocol over the air
   Flash the firmware to an nRF52840 board, scan for the advertised service, connect, discover the characteristics, subscribe to event notifications, and verify decoded payloads using a real BLE central device.
7. Complete documentation and commit final fixes
   Record actual test outcomes in the two testing documents, resolve the remaining known issues, rerun the required checks, and commit the final M2 changes.

## 7. Current challenges and risks

### 7.1 Zephyr version compatibility

Earlier compilation failures arose from differences between the project's code and the installed Zephyr API.

Examples included:

- `BT_UUID_128_ENCODE` argument-count mismatch.
- `BT_UUID_128_INIT` being unavailable.
- `BT_LE_ADV_CONN` being unavailable.
- Advertising data initializer incompatibilities.
- Missing event lifecycle implementations causing undefined references at link time.

These issues have been addressed sufficiently for a successful firmware build. Avoid reverting to the original macros without checking the installed headers.

### 7.2 Formatting corruption during code transfer

This was a recurring source of problems. Some attempted full-file replacements had multiple C statements merged onto single lines, malformed identifiers, and invalid include directives.

The established safe procedure is:

- Use `cat <<'EOF'` for complete file replacements.
- Put every include, declaration and statement on its own intended line.
- Keep paths and identifiers exactly as they appear in the repository.
- Inspect the resulting file immediately.
- Run `git diff --check`.
- Build after one logically complete change.

Prefer targeted edits when the existing file is mostly correct.

### 7.3 Build success is not runtime success

The firmware currently compiles and links, but the following remain unconfirmed:

- Whether the peripheral advertises correctly over the air.
- Whether a central can discover the expected service.
- Whether the GATT attribute index is correct at runtime.
- Whether CCC subscription state is handled correctly across disconnects.
- Whether notifications reach Android.
- Whether Android parses the notifications correctly.
- Whether reconnects and error paths work.

These need integration testing.

### 7.4 Synthetic motion data

The current firmware publishes a fixed `1800 mg` motion value at one-second intervals while notifications are enabled.

That is appropriate for a basic PoC, but it must be documented as synthetic data. Physical sensor integration is a separate requirement if M2 is intended to demonstrate real sensor acquisition.

### 7.5 Unverified integration scripts and documentation

The files were committed, but their behavior has not been demonstrated by the output seen so far.

Do not assume that a committed script works just because it exists. Inspect, run and validate it.

### 7.6 Hardware availability

No physical hardware test has been confirmed. If the nRF52840 board or Android test device is unavailable, document that limitation and complete the host-side checks. Do not mark the hardware integration portion complete without evidence.

## 8. Definition of Done for M2

M2 should be considered complete only when the agreed scope is demonstrated by repeatable tests.

| Area | Required evidence |
| --- | --- |
| Repository | Clean working tree and committed milestone |
| Firmware | Successful reproducible Zephyr build |
| Artifacts | Non-empty ELF and HEX files |
| BLE advertising | Expected device name and service UUID observed |
| GATT | Expected service and characteristics discovered |
| Connection | Central can connect and disconnect correctly |
| Notifications | Subscription works and event notifications are received |
| Protocol | C and Kotlin agree on UUIDs, payload layout and byte order |
| Code generation | Generator runs successfully and outputs are consistent |
| Python tests | Protocol tests pass |
| Android tests | Unit tests pass |
| Android build | App compiles successfully |
| Integration automation | Script runs and reports failures correctly |
| Documentation | Integration procedure and test-results record completed |
| Hardware evidence | Real BLE test results recorded, or hardware limitation explicitly documented |
| Known issues | Remaining limitations listed and accepted |

If physical hardware testing is explicitly part of the M2 acceptance criteria, a successful compile alone is insufficient.

## 9. Immediate commands for the next troubleshooting session

Start with these commands in order. They are inspection-only except for the build, and they do not modify source files.

A. Check repository state and commit

bash
```
cd ~/ssp-iot-poc && git status --short && git log -1 --oneline
```

B. Inspect the three integration deliverables

bash
```
cd ~/ssp-iot-poc && wc -lc docs/testing/ssp-ble-test-results.md docs/testing/ssp-ble-integration.md tools/test_ble_integration.sh
```

C. Inspect the integration script

bash
```
cd ~/ssp-iot-poc && sed -n '1,240p' tools/test_ble_integration.sh
```

D. Check the committed changes for whitespace errors

bash
```
cd ~/ssp-iot-poc && git diff HEAD^ HEAD --check
```

E. Verify the firmware build

bash
```
cd ~/zephyrproject && west build -b nrf52840dk/nrf52840 -d /tmp/ssp-ble-peripheral ~/ssp-iot-poc/device/firmware/ble_peripheral
```

Then paste the outputs into the troubleshooting conversation, starting with A and B.

## 10. Rules for continuing work without another troubleshooting loop

- Preserve commit `43efee7` as the current known checkpoint.
- Do not replace files wholesale unless the entire file needs replacement.
- Never make speculative Zephyr API changes without checking the installed headers.
- Never combine firmware, Android, generator and documentation fixes in one unreviewed change.
- Run the narrowest relevant test first.
- Record the first actual error rather than guessing at subsequent errors.
- Do not call an unexecuted test successful.
- Commit each verified milestone before moving to unrelated work.

Immediate objective: verify the committed checkpoint, confirm that the three previously empty testing deliverables are complete, and establish a passing baseline for firmware, protocol generation, Python tests and Android tests. Then proceed to real BLE integration and close the remaining M2 acceptance criteria.
