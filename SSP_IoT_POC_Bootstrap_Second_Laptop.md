# SSP-IoT-POC Bootstrap --- Second Windows Laptop

**Project:** SmartSecure-Perimeter (SSP)\
**Workspace:** `ssp-iot-poc` recreated on the second Windows laptop\
**Immediate goal:** Verify BLE advertising from the nRF52840 device and
detect it from the edge-side laptop software. Only after advertising is
confirmed, test connection and data exchange.

## 1. Working approach

Proceed one step at a time and keep each result. Do not troubleshoot the
complete SSP protocol until basic BLE advertising is confirmed.

**First milestone:** The nRF52840 advertises a known name and the second
laptop's BLE scanner detects it.

**Second milestone:** The SSP firmware advertises the intended SSP
name/identity and the scanner detects it.

**Third milestone:** The edge-side application connects and exchanges a
minimal test message or reads/writes a test GATT characteristic.

## 2. What has already succeeded

-   The SSP project folder `ssp-iot-poc` was recreated on the second
    Windows laptop.
-   VS Code launches from Windows Command Prompt using `code`.
-   Ubuntu is installed as a WSL 2 distribution. `wsl --list --verbose`
    showed:
    -   `Ubuntu` --- `Running`, version `2`
    -   `docker-desktop` --- `Stopped`, version `2`
-   On the **first laptop**, a native Windows Zephyr environment was
    available and a stock Zephyr Bluetooth peripheral sample built
    successfully for `nrf52840dk/nrf52840`.
-   The first laptop's J-Link could identify the nRF52840 target over
    SWD and report the target voltage. This established that the
    debugger could communicate with the MCU at that time.

These results are historical and must not be assumed to prove that the
second laptop's toolchain, USB forwarding, firmware image, or BLE scan
path works.

## 3. What has not succeeded / remains unresolved

-   On the first laptop, the scanner did **not** detect a device named
    `Zephyr` or `SSP`, even after flashing the stock Zephyr Bluetooth
    peripheral sample and scanning with Python/Bleak.
-   Other BLE devices were visible to the scanner, so the host scanner
    could detect at least some advertisements.
-   Disconnecting Proton VPN did not resolve the missing `Zephyr`/`SSP`
    advertisement.
-   The first-laptop BLE issue was left unresolved; work moved to the
    second laptop.
-   On the second laptop, attempting to launch Ubuntu with
    `wsl -d Ubuntu` produced the German error:
    `Schwerwiegender Fehler Fehlercode: WSL/Service/E_UNEXPECTED`
-   The cause of the WSL service error has not been established. Do not
    reinstall/reset Ubuntu or delete its files as a first response.
-   VS Code's `code` command works in Windows Command Prompt, but was
    not found inside Ubuntu WSL. WSL-to-VS-Code integration remains
    unverified.
-   No successful SSP BLE advertisement or edge-to-device BLE connection
    has yet been confirmed.

## 4. Important hardware / USB precautions

-   Use the actual J-Link USB bus ID reported by `usbipd list` on the
    **current laptop**. Do not reuse a bus ID from the first laptop.
-   If using the J-Link from WSL, inspect `usbipd list` and attach the
    correct device to WSL before running tools that need it. Confirm the
    device is attached before attempting a flash.
-   Avoid unplugging the board or toggling its reset/power switch
    unnecessarily. First determine whether the current test requires USB
    forwarding or a firmware reflash.
-   Do not assume that a successful build means the image was flashed or
    that advertising started.

## 5. Fastest diagnostic plan

### Step A --- Establish the current environment

On the second laptop, open Windows PowerShell or Command Prompt and run:

``` powershell
wsl --list --verbose
```

Then check the board/debugger USB visibility:

``` powershell
usbipd list
```

Record the J-Link device row and bus ID. Do not attach or detach devices
until the current setup and intended build location (Windows or WSL) are
clear.

Because WSL currently reports `Wsl/Service/E_UNEXPECTED`, do not make
WSL a dependency for the first BLE test if a native Windows Zephyr
toolchain can be used. First establish whether the second laptop already
has a working Windows-side Zephyr toolchain. If it does not, repair WSL
carefully or set up a supported native Windows toolchain; do not mix
Windows and WSL build directories or Python environments.

### Step B --- Prove the radio and scanner path with a known-good sample

1.  Build and flash the **stock Zephyr Bluetooth peripheral sample** for
    board `nrf52840dk/nrf52840`, using a known-good toolchain and the
    correct J-Link connection.
2.  Check the build and flash exit status. Preserve the exact command
    and full output.
3.  Run a BLE scan on the second laptop for 30 seconds or longer,
    displaying every device's local name, address, RSSI, and advertised
    service UUIDs---not just devices whose name contains `SSP`.
4.  Look for the stock sample's expected advertised name (commonly
    `Zephyr`, depending on sample/configuration). Do not assume the name
    without checking the sample/configuration.
5.  If no expected advertisement appears, stop before testing SSP
    application logic. Confirm the flashed image, board target, flash
    result, scanner adapter, and whether the firmware actually starts
    advertising. Use RTT/UART/logging only if configured and available.

**Pass condition:** The second laptop detects the known-good peripheral
sample repeatedly with a plausible RSSI.

### Step C --- Test SSP advertising

Only after Step B passes:

1.  Inspect the SSP project's actual source/configuration to find where
    the BLE advertising payload and local name are defined.
2.  Confirm the intended device name and any advertised service UUID. Do
    not invent or assume them; record the values from source.
3.  Build and flash the SSP firmware for the correct board.
4.  Scan for all nearby devices and compare the results with the
    expected SSP local name/UUID. A local name may be absent from some
    advertisement packets or appear in a scan response, so inspect both
    advertising data and scan response where the scanner supports it.
5.  If the stock sample is detected but SSP is not, focus on SSP
    configuration and runtime advertising start/error handling---not on
    the laptop's general BLE capability.

**Pass condition:** The second laptop detects the SSP device identity
and expected advertised service UUID, where configured.

### Step D --- Verify edge-to-device communication

After SSP advertising is confirmed:

1.  Connect from the edge-side BLE client using the discovered
    address/name and the intended service UUID.
2.  Discover GATT services and characteristics.
3.  Read a harmless test characteristic or exchange one minimal test
    message.
4.  Capture timestamps, address, RSSI, connection result,
    service/characteristic UUIDs, and any error logs.

Do not begin with encryption, pairing, telemetry, cloud connectivity, or
the full SSP workflow. Prove the smallest BLE connection/data path
first.

## 6. Scanner script for the second laptop

If Python and Bleak are available on Windows, this script lists all
detected advertisements rather than filtering only for `SSP`:

``` python
import asyncio
from bleak import BleakScanner

async def main():
    print("Scanning BLE advertisements for 30 seconds...")
    devices = await BleakScanner.discover(timeout=30, return_adv=True)

    if not devices:
        print("No BLE devices detected.")
        return

    for address, (device, adv) in devices.items():
        name = adv.local_name or device.name or "(no local name)"
        print(f"Name: {name}")
        print(f"Address: {address}")
        print(f"RSSI: {adv.rssi} dBm")
        print(f"Service UUIDs: {adv.service_uuids or 'None advertised'}")
        print("-" * 40)

asyncio.run(main())
```

Run it using the Python environment where Bleak is installed. If Bleak
is not installed, first confirm the selected Python interpreter and
install it into that environment rather than a random system Python.

## 7. Evidence log

For every test, record:

-   Laptop: second Windows laptop
-   Execution environment: Windows or WSL (do not leave ambiguous)
-   Board: `nrf52840dk/nrf52840`
-   Firmware/sample and source revision
-   Build command and result
-   Flash command and result
-   J-Link visibility/USB forwarding state
-   Scanner command, scan duration, and complete output
-   Expected versus observed local name, address, RSSI, and service
    UUIDs
-   Any UART/RTT logs or error codes

## 8. Immediate next action

**Do not reinstall anything yet.** First establish whether the second
laptop can run a known-good Zephyr BLE peripheral image and scan for it.
Since WSL has produced `Wsl/Service/E_UNEXPECTED`, check the second
laptop's Windows-side Zephyr toolchain and J-Link USB visibility before
deciding whether to repair WSL or use a native Windows build. Then
perform the stock-sample advertising test, followed by the SSP
advertising test.

The key decision is:

-   **Stock sample visible:** radio, flashing, and scanner path are
    likely functional; investigate SSP firmware/configuration next.
-   **Stock sample not visible:** stay at the basic
    firmware/advertising/debugging layer; do not debug the SSP edge
    protocol yet.
