We have identified the three remaining zero-byte files in the M2 phase of the SSP IoT project. The next step is to populate these files with useful, consistent content without claiming that firmware integration tests have passed when they have not yet run successfully.

The files are:

1. `docs/testing/ssp-ble-test-results.md` — records build and test results.
2. `docs/testing/ssp-ble-integration.md` — documents the BLE integration test procedure and expected behavior.
3. `tools/test_ble_integration.sh` — provides a repeatable integration-test script.

I recommend generating all three using `cat <<'EOF'` heredocs, as this has been the most reliable way to preserve formatting in your terminal workflow.

## 1. Generate the three files

Copy and paste the following command into your terminal. It assumes the repository is at `/home/louis/ssp-iot-poc`.

bash
````

cd ~/ssp-iot-poc || exit 1

mkdir -p docs/testing tools

cat > docs/testing/ssp-ble-test-results.md <<'EOF'
# SSP BLE Integration Test Results

## Project

SSP IoT proof of concept — M2 BLE peripheral integration.

## Test environment

- Repository: `ssp-iot-poc`
- Firmware framework: Zephyr
- Zephyr version observed: `4.5.0-rc1`
- Target board: `nrf52840dk/nrf52840`
- Firmware build directory: `/tmp/ssp-ble-peripheral`
- Build command:

  ```bash
  west build -p always \
      -b nrf52840dk/nrf52840 \
      -d /tmp/ssp-ble-peripheral \
      ~/ssp-iot-poc/device/firmware/ble_peripheral
  ```

## Results

| Check | Status | Notes |
|---|---|---|
| Zephyr configuration | Passed | CMake configuration and generation completed. |
| GATT source compilation | Passed after source fixes | The build progressed beyond compilation of `ssp_gatt.c`. |
| Advertising source | Pending verification | Confirmed advertising parameters use `BT_LE_ADV_CONN_FAST_1`. |
| Firmware linking | Failed at last recorded attempt | Undefined references to `ssp_event_init` and `ssp_event_start`. |
| Firmware image generation | Pending | A successful final link and build are required. |
| BLE advertising on hardware | Not tested | Requires a flashed board and runtime verification. |
| GATT notification delivery | Not tested | Requires a connected BLE client subscribed to notifications. |
| Full integration test | Not completed | Must be rerun after the firmware build succeeds. |

## Outstanding build issue

The last recorded linker errors were:

- Undefined reference to `ssp_event_init`
- Undefined reference to `ssp_event_start`

The inspected `src/ssp_event.c` contained the static function
`ssp_event_publish_motion()`, but the displayed file ended after that
function. The public event lifecycle functions declared in `ssp_event.h`
were not present in the inspected output.

Inspect and repair `device/firmware/ble_peripheral/src/ssp_event.c`
before attempting to validate the complete firmware.

## Test policy

Do not mark a test as passed without observing its successful result.
Update this document after each build or integration-test run.

## Final acceptance criteria

- The firmware builds and links successfully.
- The expected HEX and ELF artifacts exist.
- The peripheral advertises the SSP service.
- A BLE client can read the status characteristic.
- A BLE client can subscribe to event notifications.
- Encoded SSP events are delivered and decoded correctly.
- Results and any limitations are recorded here.
EOF

cat > docs/testing/ssp-ble-integration.md <<'EOF'
# SSP BLE Integration Test Procedure

## 1. Purpose

Verify the integration of the SSP protocol with the Zephyr BLE peripheral.

The test covers firmware compilation, BLE advertising, GATT service discovery,
status reads, event notifications, and protocol payload handling.

## 2. Prerequisites

- Repository checked out at `~/ssp-iot-poc`.
- Zephyr workspace available at `~/zephyrproject`.
- The Zephyr Python environment activated.
- Zephyr SDK and required host tools installed.
- An nRF52840 DK for hardware tests.
- A BLE-capable client for discovery and notification tests.

Hardware tests must not be reported as completed when only a host-side
firmware build has been performed.

## 3. Build the firmware

Run the following command as one line:

```bash
cd ~/zephyrproject && west build -p always -b nrf52840dk/nrf52840 -d /tmp/ssp-ble-peripheral ~/ssp-iot-poc/device/firmware/ble_peripheral
```

The build must finish successfully, including linking.

Verify the generated artifacts:

```bash
ls -lh /tmp/ssp-ble-peripheral/zephyr/zephyr.hex /tmp/ssp-ble-peripheral/zephyr/zephyr.elf
```

If compilation or linking fails, record the first relevant error and fix it
before proceeding.

## 4. Inspect the firmware sources

Confirm that:

- `ssp_gatt.c` uses UUID initializers supported by the checked-out Zephyr
  version.
- `ssp_gatt_notify()` validates the payload and active connection.
- `ssp_advertising.c` includes flags, the 128-bit SSP service UUID, and the
  complete configured device name.
- Advertising uses a connection-oriented advertising parameter supported by
  the checked-out Zephyr version.
- `ssp_event.c` implements every public lifecycle function declared in
  `ssp_event.h`.
- The application calls the event initialization and start functions only
  after their implementations are linked into the firmware.

## 5. Flash and start the peripheral

Connect the nRF52840 DK and use the project's established flashing workflow.

After flashing, open the appropriate serial console and reset the board.

Expected behavior includes startup messages indicating that the application
and Bluetooth stack initialized and that advertising started.

Record any errors and the board used.

## 6. Verify BLE advertising

Using a BLE scanner:

1. Scan for the configured SSP device name.
2. Confirm that the peripheral is discoverable.
3. Discover its 128-bit SSP service UUID.
4. Record the device name and observed service UUID.

Compare the observed UUID with `SSP_SERVICE_UUID_STR` and the UUID used in
the GATT service definition.

## 7. Verify the GATT status characteristic

Connect to the peripheral and discover the SSP service.

Read the status characteristic and confirm that the returned value matches
the expected protocol status.

Record the UUID, returned value, and result.

## 8. Verify event notifications

1. Discover the SSP event characteristic.
2. Enable notifications through the client's CCC subscription.
3. Trigger or wait for an event using the implemented event mechanism.
4. Capture the notification payload.
5. Decode it with the SSP protocol implementation.
6. Verify protocol version, event type, status, device ID, sequence number,
   timestamp, and motion value as applicable to the event definition.

Confirm that notification attempts without an active connection or without
a subscription are handled as expected.

## 9. Record results

For each test, record:

- Date and test environment.
- Firmware revision.
- Build result.
- Hardware and client used.
- Expected and observed behavior.
- Pass, fail, or blocked status.
- Relevant logs and follow-up actions.

Update `docs/testing/ssp-ble-test-results.md`.

## 10. Completion criteria

The integration is accepted only when the firmware builds successfully and
the required BLE behaviors have been observed.

A successful compilation alone does not prove that advertising, GATT reads,
or notifications work on physical hardware.
EOF

cat > tools/test_ble_integration.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

REPO_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
BUILD_DIR="${BUILD_DIR:-/tmp/ssp-ble-peripheral}"
BOARD="${BOARD:-nrf52840dk/nrf52840}"
APP_DIR="${REPO_ROOT}/device/firmware/ble_peripheral"

echo "SSP BLE integration test"
echo "Repository: ${REPO_ROOT}"
echo "Board:      ${BOARD}"
echo "Build dir:  ${BUILD_DIR}"
echo

if ! command -v west >/dev/null 2>&1; then
    echo "ERROR: west is not available on PATH." >&2
    echo "Activate the project's Zephyr environment and retry." >&2
    exit 1
fi

if [[ ! -f "${APP_DIR}/CMakeLists.txt" ]]; then
    echo "ERROR: Firmware application not found: ${APP_DIR}" >&2
    exit 1
fi

echo "Checking required event lifecycle declarations..."
if ! grep -q 'ssp_event_init' "${APP_DIR}/src/ssp_event.h"; then
    echo "ERROR: ssp_event_init is not declared in ssp_event.h." >&2
    exit 1
fi

if ! grep -q 'ssp_event_start' "${APP_DIR}/src/ssp_event.h"; then
    echo "ERROR: ssp_event_start is not declared in ssp_event.h." >&2
    exit 1
fi

echo "Running firmware build..."
west build -p always \
    -b "${BOARD}" \
    -d "${BUILD_DIR}" \
    "${APP_DIR}"

HEX_FILE="${BUILD_DIR}/zephyr/zephyr.hex"
ELF_FILE="${BUILD_DIR}/zephyr/zephyr.elf"

if [[ ! -s "${HEX_FILE}" ]]; then
    echo "ERROR: Firmware HEX artifact is missing or empty: ${HEX_FILE}" >&2
    exit 1
fi

if [[ ! -s "${ELF_FILE}" ]]; then
    echo "ERROR: Firmware ELF artifact is missing or empty: ${ELF_FILE}" >&2
    exit 1
fi

echo
echo "PASS: Firmware build and link completed."
echo "Generated artifacts:"
ls -lh "${HEX_FILE}" "${ELF_FILE}"

echo
echo "NOTE: This script verifies compilation, linking, and artifacts only."
echo "Physical BLE advertising, GATT reads, and notifications require"
echo "separate hardware integration tests."
EOF

chmod +x tools/test_ble_integration.sh

echo
echo "Created files:"
wc -l \
    docs/testing/ssp-ble-test-results.md \
    docs/testing/ssp-ble-integration.md \
    tools/test_ble_integration.sh

echo
echo "Zero-byte file check:"
./tools/list_empty_files.sh
````

## 2. What these files accomplish

- The results document preserves the actual state of the project: the GATT compile errors were addressed, but the last build failed during linking because event lifecycle functions were unresolved.
- The integration guide defines the steps for verifying the firmware and BLE behavior.
- The shell script provides a repeatable build-and-artifact check. It does not claim to test a physical BLE connection.

## 3. Resume the current troubleshooting

Once the files are created, the next priority remains `ssp_event.c`. The last build showed that `main.c` calls `ssp_event_init()` and `ssp_event_start()`, while the inspected event source only contained `ssp_event_publish_motion()`.

Run this command to inspect the current implementation:

bash
```
cd ~/ssp-iot-poc && nl -ba device/firmware/ble_peripheral/src/ssp_event.c
```

We should fix the missing event lifecycle implementations before treating the firmware build as complete.
