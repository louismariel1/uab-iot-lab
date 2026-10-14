# Project Bootstrap — SSP IoT Proof of Concept (BLE Firmware)

**Bootstrap date:** 14 October 2026\
**Project repository:** `~/ssp-iot-poc`\
**Development environment:** Linux, Zephyr RTOS, Python 3.12 virtual environment\
**Current status:** Firmware compilation has progressed from C compilation errors to a linker error. A proposed fix for the linker error has been prepared but its build result is not yet known.

## 1\. Project goal

Develop an IoT proof of concept (SSP IoT POC) featuring a Bluetooth Low Energy (BLE) peripheral implemented using Zephyr RTOS on the Nordic nRF52840 development board.

The current firmware provides the foundation for:

- BLE initialization and advertising.
- An SSP GATT service with status and event characteristics.
- Notification-based transmission of encoded SSP events.
- A Python-based project workflow and eventual integration with Android.
- A repeatable build process that can be extended toward hardware testing.

The target board is `nrf52840dk/nrf52840`.

## 2\. Development environment

- Repository: `~/ssp-iot-poc`
- Zephyr workspace: `~/zephyrproject`
- Zephyr source: `~/zephyrproject/zephyr`
- Zephyr version detected by the build: `4.5.0-rc1`, build `v4.5.0-rc1-43-ga90cd40a5f0a`
- Python environment: `zephyr-3.12`, using Python 3.12.15
- Zephyr SDK: `~/zephyr-sdk-1.0.1`
- Dedicated firmware build directory: `/tmp/ssp-ble-peripheral`
- Firmware application source: `~/ssp-iot-poc/device/firmware/ble_peripheral`

Activate the existing Python environment if necessary, then use the Zephyr workspace for builds.

**Important:** Run `west build` as one physical terminal line to avoid shell interpreting continuation lines as separate commands.

## 3\. What has been achieved

### A. Advertising source corrected

File:

`device/firmware/ble_peripheral/src/ssp_advertising.c`

The file has been replaced with correctly formatted C using a `cat <<'EOF'` heredoc.

The current implementation:

- Parses the canonical service UUID string from `SSP_SERVICE_UUID_STR`.
- Converts the UUID to little-endian bytes for advertising.
- Uses an explicit flags array instead of `BT_DATA_BYTES`.
- Advertises the 128-bit service UUID and complete device name.
- Uses `BT_LE_ADV_CONN_FAST_1` instead of the unavailable `BT_LE_ADV_CONN` macro.

A grep check confirmed the advertising source contains `BT_LE_ADV_CONN_FAST_1` and no obsolete `BT_DATA_BYTES` or `advertising_flags` references.

### B. GATT source corrected

File:

`device/firmware/ble_peripheral/src/ssp_gatt.c`

The file was replaced completely using a heredoc after an earlier malformed copy had been appended to it.

The latest implementation includes:

- The required standard headers.
- Three SSP UUID declarations using designated initializers and `BT_UUID_TYPE_128`, rather than the unavailable `BT_UUID_128_INIT` macro.
- A readable status characteristic.
- A notification-enabled event characteristic and CCC descriptor.
- Connection reference management.
- A null-connection guard returning `-ENOTCONN` in `ssp_gatt_notify()`.
- Notification delivery through `bt_gatt_notify()` using `&ssp_service.attrs[4]`.

The verification command confirmed all three UUID declarations use `BT_UUID_TYPE_128` and found no malformed lines containing multiple `#include` directives.

**Note:** The current source compiled successfully in the most recent build. The attribute index and runtime BLE behavior have not yet been tested on hardware.

### C. First successful C compilation of the corrected GATT source

The build previously failed on `BT_UUID_128_INIT`, producing three invalid-initializer errors. After replacing the GATT file with the designated-initializer implementation, the next build compiled `ssp_gatt.c` successfully and progressed to the final linking stage.

This confirms the earlier UUID compilation problem is resolved.

## 4\. Current problem

The latest completed build failed during linking, not C compilation.

Build command:

`west build -p always -b nrf52840dk/nrf52840 -d /tmp/ssp-ble-peripheral ~/ssp-iot-poc/device/firmware/ble_peripheral`

The relevant linker errors were:

```
undefined reference to `ssp_event_init'
undefined reference to `ssp_event_start'
```

Both references originate from `device/firmware/ble_peripheral/src/main.c`.

The build also reported:

```
warning: 'ssp_event_publish_motion' defined but not used
```

### Relevant event files before the proposed fix

`device/firmware/ble_peripheral/src/ssp_event.h` declares:

```
void ssp_event_init(void);
void ssp_event_start(void);
bool ssp_event_subscribed(void);
```

However, the previous `ssp_event.c` contained only a private function:

```
static int ssp_event_publish_motion(void)
```

That explains why the linker could not find the public functions.

### Proposed fix prepared but not yet verified

A complete replacement for `device/firmware/ble_peripheral/src/ssp_event.c` was provided using a heredoc.

The proposed implementation:

- Includes `ssp_event.h` and `ssp_gatt.h`.
- Retains the existing SSP event encoding.
- Defines a Zephyr timer that fires every 1,000 milliseconds.
- Implements `ssp_event_init()`, `ssp_event_start()`, and `ssp_event_subscribed()`.
- Publishes synthetic motion events only when GATT notifications are enabled.
- Logs notification errors.

The event uses the existing test value of `1800 mg`; it does not read a physical motion sensor.

**The user has not yet supplied confirmation that this file replacement was executed or that a subsequent build succeeded. Resume by checking the actual file and build state rather than assuming the fix is installed.**

## 5\. Immediate next steps

### Step 1 — Inspect the event source

Run:

```
cd ~/ssp-iot-poc

nl -ba device/firmware/ble_peripheral/src/ssp_event.c | sed -n '1,200p'

echo "=== Event interface ==="
cat device/firmware/ble_peripheral/src/ssp_event.h

echo "=== Event symbol definitions ==="
grep -nE '^(void|bool) ssp_event_(init|start|subscribed)' \
    device/firmware/ble_peripheral/src/ssp_event.c
```

Confirm that all three public functions exist and that the file is properly formatted.

### Step 2 — Rebuild the firmware

Execute this single-line command:

```
cd ~/zephyrproject && west build -p always -b nrf52840dk/nrf52840 -d /tmp/ssp-ble-peripheral ~/ssp-iot-poc/device/firmware/ble_peripheral
```

If it fails, capture the first actual compiler or linker error and its surrounding output. Do not change unrelated source files preemptively.

### Step 3 — Verify the build artifacts

Only if the build succeeds:

```
ls -lh \
    /tmp/ssp-ble-peripheral/zephyr/zephyr.hex \
    /tmp/ssp-ble-peripheral/zephyr/zephyr.elf
```

### Step 4 — Run the repository workflow

After the standalone firmware build succeeds, return to the repository and inspect or execute the project's established workflow:

```
cd ~/ssp-iot-poc
bash tools/build_ble_demo.sh
```

This workflow may also run Android unit tests. Address any subsequent Android or Gradle failures separately from the firmware build.

## 6\. Remaining work

### Immediate priorities

1. Confirm the event source replacement is installed correctly.
2. Resolve any remaining linker or compiler errors.
3. Confirm successful generation of the ELF and HEX artifacts.
4. Run the full project workflow.
5. Address any Android or Python workflow failures separately.

### Subsequent technical validation

After a successful build:

- Confirm BLE advertising and service discovery.
- Verify the SSP service and characteristic UUIDs against the host-side protocol definition.
- Verify the GATT attribute index used for event notifications.
- Test connection establishment and notification subscription.
- Confirm event encoding and sequence-number behavior.
- Test firmware on the nRF52840 development board.
- Replace synthetic motion events with real sensor data if required by the project specification.

## 7\. Important troubleshooting lessons

- Always distinguish compile errors from linker errors. The current blocker is a missing event implementation, not the earlier UUID macro problem.
- Use the exact source paths with underscores: `ble_peripheral`, `ssp_advertising.c`, and `ssp_gatt.c`.
- When providing replacement C source, use a complete `cat > path <<'EOF'` heredoc with normal line breaks.
- Avoid appending a replacement source file to an existing file.
- Verify files using `nl`, `grep`, and `tail` before rebuilding.
- Run the build command on one terminal line to prevent shell line-continuation mistakes.
- Fix the first actionable error, rebuild, and inspect the next error. Do not make unrelated changes.

## 8\. Current project status summary

| Component | Status |
| --- | --- |
| Zephyr environment and SDK | Detected and working |
| Advertising source | Updated; C compilation progressed successfully |
| GATT UUID initializers | Corrected; GATT source compiled |
| GATT connection guard | Added |
| Event implementation | Replacement proposed; execution/build confirmation pending |
| Standalone firmware build | Not yet successful |
| ELF/HEX artifact verification | Pending |
| Full project workflow | Pending |
| Android integration and hardware validation | Pending |

**Resume instruction:** Start with Section 5, Step 1. Inspect the current `ssp_event.c`, verify the missing functions are present, then run the standalone firmware build. Do not repeat the already-resolved UUID macro troubleshooting unless a new error specifically requires it.
