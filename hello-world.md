## 1. Create a permanent environment document

I recommend adding `docs/testing/development-environment.md` to your existing `ssp-iot-poc` repository. It will record the verified configuration, explain the Python environment, and provide recovery instructions.

Run this from any directory in Ubuntu:

bash
```
mkdir -p ~/ssp-iot-poc/docs/testing
cd ~/ssp-iot-poc
```

Create the document:

Writing

# SSP PoC — Development Environment and Firmware Smoke Test

## 1\. Purpose

This document records the development environment used for the SmartSecurePerimeter (SSP) IoT proof of concept.

The purpose is to make the environment reproducible, prevent unnecessary reinstallation of tools, and provide a known procedure for verifying the nRF52840-DK hardware and firmware toolchain.

The initial hardware validation covers:

- Windows USB access and USB passthrough to WSL2.
- SEGGER J-Link detection.
- Zephyr workspace and SDK availability.
- Firmware compilation and flashing.
- UART output reception through the J-Link serial interface.

A successful USB detection test alone does not prove that firmware flashing and UART communication work. Those must be validated separately.

## 2\. Development architecture

The current development environment is organized as follows:

```
Windows 10
    |
    | USB
    v
usbipd-win
    |
    | USB/IP
    v
WSL2 / Ubuntu
    |
    +-- pyenv
    |     |
    |     +-- Python 3.12.15
    |     +-- zephyr-3.12 virtual environment
    |     +-- west 1.5.0
    |
    +-- Zephyr workspace
    |     /home/louis/zephyrproject
    |
    +-- Zephyr SDK
    |     /home/louis/zephyr-sdk-1.0.1
    |
    +-- SSP project
          /home/louis/ssp-iot-poc
               |
               +-- device/
               +-- edge/
               +-- cloud/
               +-- ml/
               +-- docs/
```

The Windows host provides USB connectivity. WSL2 is the primary firmware development environment. The nRF52840-DK is the target device.

## 3\. Verified configuration

The following components have been observed in the development environment.

| Component | Configuration or observation |
| --- | --- |
| Host operating system | Windows 10, as previously recorded |
| Linux environment | WSL2 / Ubuntu |
| USB forwarding | usbipd-win |
| J-Link USB identifier | `1366:1025` |
| J-Link serial interface | `/dev/ttyACM0` |
| Stable serial identifier | `/dev/serial/by-id/usb-SEGGER_J-Link_000683321542-if00` |
| Python version for Zephyr | Python 3.12.15 |
| Python environment | `zephyr-3.12` |
| West version | 1.5.0 |
| Zephyr workspace | `/home/louis/zephyrproject` |
| Zephyr SDK directory | `/home/louis/zephyr-sdk-1.0.1` |
| SSP project | `/home/louis/ssp-iot-poc` |

The J-Link USB device and serial interface have been detected successfully in WSL2.

**Remaining validation:** Repeat the firmware build, flashing and UART reception tests. These capabilities must not be considered verified until the smoke test passes again.

## 4\. Python environment management

### 4.1 Why the virtual environment matters

The system contains several Python installations managed through pyenv.

The default shell configuration previously selected Python 3.14.4. However, the `west` command is installed in the Python 3.12.15 environment named `zephyr-3.12`.

Therefore, the correct Python environment must be activated before executing Zephyr commands.

### 4.2 Activate the Zephyr environment

Open Ubuntu/WSL and run:

```
pyenv versions
pyenv activate zephyr-3.12
```

Verify the active environment:

```
python --version
python -c "import sys; print(sys.executable)"
west --version
```

Expected results:

```
Python 3.12.15
/home/louis/.pyenv/versions/zephyr-3.12/bin/python
West version: v1.5.0
```

The exact output formatting may differ slightly.

If `pyenv activate` is unavailable because shell initialization is missing, investigate the pyenv configuration before installing another Python version.

### 4.3 Important distinction

Activating a Python environment and entering a Zephyr workspace are separate operations.

Activating `zephyr-3.12` makes the appropriate Python tools available. It does not make every directory a Zephyr workspace.

The Zephyr workspace currently resides at:

```
/home/louis/zephyrproject
```

West commands such as `west build` and `west flash` must be executed in a context where West can locate that workspace.

### 4.4 Python environment responsibilities

| Environment | Responsibility |
| --- | --- |
| `zephyr-3.12` | Zephyr tooling and `west` |
| System Python | Operating-system utilities and any system-installed Python packages |
| SSP project environment | Future backend, testing, analytics and ML dependencies |

Do not install all project dependencies into the Zephyr environment. Keep firmware tooling separate from application-level dependencies.

The smoke-test script uses `python3` to import `serial`. Verify that this interpreter can import pySerial before running the test:

```
python3 -c "import sys; print(sys.executable); import serial; print(serial.__version__)"
```

If the active `python3` cannot import `serial`, resolve that dependency explicitly rather than assuming that activation of `zephyr-3.12` makes it available.

## 5\. Zephyr workspace and SDK

The Zephyr workspace is:

```
/home/louis/zephyrproject
```

The SDK directory is:

```
/home/louis/zephyr-sdk-1.0.1
```

After activating `zephyr-3.12`, verify the workspace:

```
cd ~/zephyrproject
west topdir
west list zephyr
```

The expected workspace root is:

```
/home/louis/zephyrproject
```

The SDK directory can be checked with:

```
ls -ld ~/zephyr-sdk-1.0.1
```

Do not reinstall the SDK or recreate the workspace unless diagnostics demonstrate that the existing installation is unusable.

The exact Zephyr revision and toolchain configuration should be recorded from the actual installation when the environment is next validated.

## 6\. SSP firmware source and build directories

The SSP Hello World application source is maintained inside the Git repository:

```
~/ssp-iot-poc/device/firmware/hello_world/
├── CMakeLists.txt
├── prj.conf
└── src/
    └── main.c
```

The previous smoke-test script expects the compiled firmware at:

```
/tmp/ssp-hello-world/
└── zephyr/
    ├── zephyr.hex
    └── zephyr.elf
```

These locations serve different purposes:

- The source directory is part of the SSP Git repository.
- The Zephyr workspace contains the Zephyr source and modules.
- The build directory contains generated files and firmware images.
- The temporary build directory under `/tmp` is not permanent project storage.

The temporary build directory was missing during the latest validation attempt. It must be recreated by a successful build before the smoke-test script can run.

## 7\. USB passthrough procedure

Connect the nRF52840-DK to the Windows computer.

In Windows PowerShell:

```
usbipd list
```

Identify the current BUSID for the SEGGER J-Link device with USB identifier `1366:1025`.

Attach the device to WSL using the current BUSID:

```
usbipd attach --wsl --busid <BUSID>
```

Replace `<BUSID>` with the actual value reported by `usbipd list`.

Do not assume that the BUSID remains constant after reconnecting the board.

In Ubuntu/WSL, verify USB detection:

```
lsusb
```

Expected entry:

```
ID 1366:1025 SEGGER J-Link
```

Verify the serial interface:

```
ls -l /dev/ttyACM*
ls -l /dev/serial/by-id/
```

Previously observed devices:

```
/dev/ttyACM0
usb-SEGGER_J-Link_000683321542-if00 -> ../../ttyACM0
```

If the device is not detected, investigate USB passthrough before attempting to flash firmware.

## 8\. PCB Flash and UART Smoke Test

### 8.1 Objective

The smoke test verifies the end-to-end hardware path:

```
Windows USB
    |
    v
usbipd-win
    |
    v
WSL2 / Ubuntu
    |
    v
SEGGER J-Link
    |
    v
nRF52840-DK
    |
    v
Zephyr firmware
    |
    v
UART
    |
    v
J-Link USB serial interface
    |
    v
Python / pySerial
```

A successful test requires both successful firmware flashing and reception of the expected UART message.

### 8.2 Existing test script

The script is located at:

```
~/flash_hello_test.sh
```

It currently expects:

```
Build directory: /tmp/ssp-hello-world
Serial device:   /dev/ttyACM0
Baud rate:       115200
Timeout:         15 seconds
Expected text:   Hello World! nrf52840dk/nrf52840
```

The script:

1. Checks that the firmware HEX file exists.
2. Checks that the serial device exists.
3. Starts a Python serial listener.
4. Runs `west flash` using the J-Link runner.
5. Waits for the expected UART message.
6. Reports PASS or FAIL.

The script currently assumes that `west` is available from its execution context. Before using it, ensure that the Zephyr environment is activated and that the script can locate the Zephyr workspace.

### 8.3 Build the firmware

First activate the correct Python environment:

```
pyenv activate zephyr-3.12
```

Then enter the Zephyr workspace:

```
cd ~/zephyrproject
```

Build the SSP Hello World application into the directory expected by the test script:

```
west build -p always \
  -b nrf52840dk/nrf52840 \
  -d /tmp/ssp-hello-world \
  ~/ssp-iot-poc/device/firmware/hello_world
```

This command builds firmware but does not flash the board.

If the build fails, preserve the complete error output for diagnosis.

### 8.4 Verify build artifacts

After a successful build:

```
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.hex
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.elf
```

Do not proceed until the HEX file exists.

### 8.5 Flash and verify UART

Ensure the J-Link is attached to WSL and the serial device is present.

Activate the Zephyr environment, then execute the smoke-test script:

```
pyenv activate zephyr-3.12
~/flash_hello_test.sh
```

**Important:** The script's `west flash` command must execute in a context where the Zephyr workspace is discoverable. If it fails with `unknown command "flash"` or a workspace error, correct the script's workspace handling before interpreting the result as a hardware failure.

Expected successful output includes:

```
EXPECTED MESSAGE RECEIVED

========================================
  nRF52840 Hello World TEST: PASS
========================================

Flash succeeded and the expected UART
message was received.
```

A PASS confirms that the flash operation succeeded and the expected message was received over UART during the test.

A FAIL must be investigated using the actual error output. It should not automatically be attributed to faulty hardware.

## 9\. Troubleshooting guide

| Symptom | Initial investigation |
| --- | --- |
| J-Link missing from `lsusb` | Check Windows `usbipd list` and attach the correct BUSID |
| `/dev/ttyACM0` missing | Check USB passthrough and serial-device enumeration |
| `west: command not found` | Activate `zephyr-3.12` and verify `which west` |
| `west: unknown command "build"` | Check whether the command is being run inside the Zephyr workspace |
| Firmware HEX missing | Build the application into the expected directory |
| Flashing fails | Inspect the complete `west flash` output and J-Link detection |
| Flash succeeds but UART times out | Check serial-device selection, baud rate, console configuration and expected firmware output |
| Python cannot import `serial` | Check which Python interpreter runs the listener and whether pySerial is installed there |

## 10\. Validation status

Status at the time this document was created:

- [x]Windows detects the SEGGER J-Link.
- [x]USB device is shared and attached to WSL2.
- [x]WSL detects USB identifier `1366:1025`.
- [x]`/dev/ttyACM0` exists.
- [x]Stable J-Link serial-device identifier exists.
- [x]Python 3.12.15 environment `zephyr-3.12` can be activated.
- [x]West 1.5.0 is available in that environment.
- [x]Zephyr workspace exists.
- [x]Zephyr SDK directory exists.
- [ ]SSP Hello World build recreated in `/tmp/ssp-hello-world`.
- [ ]Firmware flashing verified during the current test.
- [ ]Expected UART message received during the current test.
- [ ]Complete flash-and-UART smoke test passed during the current test.

Update the validation checklist only when the corresponding test has actually succeeded.

## 11\. Maintenance policy

- Preserve the existing Zephyr workspace and SDK unless a specific problem requires changes.
- Activate `zephyr-3.12` before using Zephyr tools.
- Keep generated firmware build files out of the source repository.
- Record changes to toolchain versions and build procedures.
- Run the smoke test before debugging application-level BLE or sensor problems.
- Keep test evidence and environment documentation in the SSP repository.

## 2. Save it in Git

You can create the file directly from the terminal using your preferred editor. For example, with VS Code:

bash
```
cd ~/ssp-iot-poc
code docs/testing/development-environment.md
```

Paste the document above and save it.

Then review the changes:

bash
```
git status
git diff --check
git diff -- docs/testing/development-environment.md
```

Because this is a new file, `git diff` will not show its contents until it is staged. To review the staged document:

bash
```
git add docs/testing/development-environment.md
git diff --cached --check
git diff --cached -- docs/testing/development-environment.md
```

If everything looks correct, commit it:

bash
```
git commit -m "docs: record Zephyr development environment"
```

## 3. One important correction before the next smoke test

Your earlier build command failed because it was executed from the SSP application directory, not from the Zephyr workspace. The documented build command fixes that by explicitly running `west build` from `~/zephyrproject` while passing the SSP source directory as the application path.

After saving the document, return to the actual test:

bash
```
pyenv activate zephyr-3.12
cd ~/zephyrproject

west build -p always \
  -b nrf52840dk/nrf52840 \
  -d /tmp/ssp-hello-world \
  ~/ssp-iot-poc/device/firmware/hello_world
```

Do not flash yet if the build fails.

One additional issue remains: the smoke-test script itself invokes `west flash` without changing to the Zephyr workspace. We should correct that script to locate the workspace reliably before depending on it as a permanent automated test. That correction should be documented and committed separately so the history clearly distinguishes environment documentation from test-script changes.
