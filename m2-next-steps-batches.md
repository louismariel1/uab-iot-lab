This additional information improves the plan in an important way. We now have a documented, automated hardware smoke test—not just a successful Hello World flash report.

There is also a specific version detail worth preserving: the UART example reports a Zephyr build identified as `v4.5.0-rc1-43-ga90cd40a5f0a`. That is evidence of the firmware image's build identity, not yet definitive proof of the currently active workspace version. We should verify the installed workspace before generating BLE code.

I would update the previous plan in four ways:

- Preserve `flash_hello_test.sh` as a permanent, independent hardware smoke test.
- Record the complete Windows USB → WSL → J-Link → nRF52840 → UART procedure in project documentation.
- Run the smoke test before investigating application-level hardware failures, but do not run it unnecessarily after every source-only change.
- Use separate build directories for Hello World, the BLE probe and the SSP application so the known-good image cannot be accidentally confused with a new build.

## 1. Add a permanent hardware test document

Create `docs/testing/pcb-flash-smoke-test.md` in the repository. This turns the recovered procedure into project documentation and prevents it from being lost again.

Writing

# PCB Flash and UART Smoke Test

## Purpose

Verify the complete hardware programming and UART diagnostic path before investigating SSP application-level failures.

The test uses a known-good Zephyr Hello World firmware image and the SEGGER J-Link interface.

## Hardware and environment

- Host: Windows with WSL 2 and Ubuntu.
- USB forwarding: `usbipd`.
- Development environment: Python virtual environment `zephyr-3.12`.
- RTOS: Zephyr.
- Build system: `west` and CMake.
- Debug/programming interface: SEGGER J-Link.
- Serial output: J-Link USB serial interface, typically `/dev/ttyACM0`.
- Expected UART message: `Hello World! nrf52840dk/nrf52840`.

Confirm the actual hardware target and firmware image configuration before running the test.

## Procedure

### 1\. Attach the J-Link from Windows

In Windows PowerShell:

```
usbipd list
usbipd attach --wsl --busid <CURRENT_BUSID>
```

Use the BUSID reported by `usbipd list`. Do not assume it remains constant.

### 2\. Activate the WSL development environment

```
pyenv activate zephyr-3.12

python --version
python -c "import sys; print(sys.executable)"
west --version
python -c "import serial; print(serial.__version__)"
```

### 3\. Verify USB and serial access

```
lsusb
ls -l /dev/ttyACM0
ls -l /dev/serial/by-id/
```

Check the stable serial-device identifier where available.

### 4\. Verify the Zephyr workspace

```
cd ~/zephyrproject
west topdir
west list
```

Record the active Zephyr revision when investigating environment problems.

### 5\. Verify the Hello World firmware image

```
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.hex
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.elf
```

These files must belong to the intended Hello World build.

### 6\. Run the automated smoke test

```
~/flash_hello_test.sh
```

The script is expected to check the firmware image and serial device, start a UART listener, flash the firmware, and confirm receipt of the expected UART message.

### 7\. Acceptance criterion

Expected result:

```
========================================
  nRF52840 Hello World TEST: PASS
========================================

Flash succeeded and the expected UART
message was received.
```

A PASS establishes that the tested flash-and-UART path worked under the conditions of that run.

A FAIL should trigger investigation of the specific failing stage before application-level debugging.

## Scope and limitations

This is a hardware and development-environment smoke test. It does not validate:

- Accelerometer/IMU acquisition.
- SSP motion detection.
- BLE advertising or GATT operation.
- BLE notifications.
- Android communication.
- End-to-end SSP behavior.

Keep this test independent of the SSP application. Do not change its expected UART message merely to accommodate a different application.

When testing another firmware image, use a separate application-specific test or an explicitly configured generalized test harness.

## Reproducibility record

For each significant hardware test, record:

- Date and time.
- Git commit.
- Zephyr revision.
- Board and hardware revision.
- Build directory and firmware image.
- Flash result.
- UART device and observed output.
- Final PASS/FAIL result.
- Any relevant error logs.

## 2. Update the batch-generation plan

The smoke test should become Gate 0 in the implementation pipeline.

| Gate | Test | When to run |
| --- | --- | --- |
| G0 | Existing Hello World flash/UART smoke test | Before hardware debugging, or when the baseline is uncertain |
| G1 | SSP firmware compilation | After generating or modifying firmware |
| G2 | Synthetic motion unit tests | After changing the motion-processing logic |
| G3 | BLE peripheral verification | After generating the GATT service and advertising implementation |
| G4 | Android build and parser tests | After generating or modifying the Edge application |
| G5 | Device-to-Edge integration | After both endpoints are ready |
| G6 | Physical IMU acceptance test | Once the actual sensor is identified and integrated |

This arrangement avoids repeatedly running a hardware flash test when only an Android parser has changed.

### One important safeguard for the existing script

Your `flash_hello_test.sh` is specifically designed to validate Hello World. Do not reuse it unchanged for the SSP BLE application. Its expected UART string and firmware image are application-specific.

Later, we can create a shared test harness with separate configurations for Hello World and SSP, or simply retain the current script and add `flash_ssp_ble_test.sh`. I favor retaining the known-good script and adding a separate SSP test initially, because that minimizes the risk of damaging a working diagnostic tool.

## 3. Revised immediate priorities

Here is the updated order of work:

1. Commit the recovered `device/` baseline after inspecting it.
2. Add `docs/testing/pcb-flash-smoke-test.md`.
3. Audit the active Zephyr workspace and available Bluetooth samples.
4. Run the existing Hello World smoke test if the current hardware environment needs verification.
5. Freeze the SSP BLE protocol and shared test vectors.
6. Generate the firmware batch and Android batch against the same contract.
7. Automate builds, unit tests and targeted repairs.
8. Validate the complete synthetic-event-to-Android path.
9. Integrate the physical IMU and complete the M2 acceptance tests.

One final technical distinction: the procedure names the target `nrf52840dk/nrf52840`, while the documentation also refers to a PCB. We should confirm whether the tested hardware is the Nordic development kit or a custom PCB using an nRF52840. That affects what we can infer from a passing smoke test and which board configuration should be used for the next firmware build.

The strategy is now clear: preserve the proven baseline, formalize the hardware test, audit the real toolchain, then generate the largest coherent batches that can still be tested and debugged independently.
