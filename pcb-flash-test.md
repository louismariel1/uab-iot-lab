# PCB Flash and UART Smoke Test

 ## 1\. Intent and usefulness of the test

 This test verifies that the PCB can be successfully programmed through the SEGGER J-Link interface and that the programmed firmware boots and produces its expected UART output.

 The test validates the complete development and hardware path:

```
Windows USB
    ↓
usbipd
    ↓
WSL
    ↓
J-Link
    ↓
nRF52840
    ↓
Zephyr firmware
    ↓
UART
    ↓
J-Link USB Serial
    ↓
/dev/ttyACM0
```

 The test uses a minimal Zephyr `Hello World` application as a known-good firmware image.

 A successful test demonstrates that:

 - The J-Link is correctly connected to the Windows host.
- The J-Link USB device can be passed through to WSL.
- WSL can access the J-Link.
- `west flash` can program the nRF52840.
- The nRF52840 successfully boots the programmed firmware.
- The J-Link USB serial interface is accessible as `/dev/ttyACM0`.
- UART output from the PCB can be received by WSL.
- The Python/pyserial environment is functioning correctly.

 This is intended as a **hardware and development-environment smoke test**. It should be run before starting application-level debugging if there is any uncertainty about the development environment or PCB connection.

 The test script automatically reports `PASS` only when both flashing and receipt of the expected UART message succeed.

---

 ## 2\. Windows: Connect the PCB to WSL

 Connect the PCB/J-Link USB cable to the Windows computer.

 Open **Windows PowerShell** and list the USB devices:

```
usbipd list
```

 Identify the J-Link device. For example:

```
Connected:
BUSID  VID:PID    DEVICE
1-4    1366:1025  USB Serial Device (COM3), BULK interface, USB Mass Storag...
```

 The BUSID may be different on another computer or after reconnecting the device. Do **not** assume it will always be `1-4`; use the BUSID reported by `usbipd list`.

 Attach the J-Link USB device to WSL:

```
usbipd attach --wsl --busid 1-4
```

 A successful attachment looks similar to:

```
usbipd: info: Using WSL distribution 'Ubuntu' to attach; the device will be available in all WSL 2 distributions.
usbipd: info: Detected networking mode 'nat'.
usbipd: info: Using IP address ... to reach the host.
```

 If Windows reports that the device is busy, make sure that no Windows application is currently using the J-Link serial port, such as a Python serial listener.

---

 ## 3\. WSL: Verify the J-Link serial device

 Open a WSL terminal.

 Verify that the serial device is available:

```
ls -l /dev/ttyACM0
```

 The expected result is similar to:

```
crw-rw-rw- 1 root dialout ... /dev/ttyACM0
```

 It is also useful to verify the stable device name:

```
ls -l /dev/serial/by-id/
```

 For this J-Link, the expected entry is similar to:

```
usb-SEGGER_J-Link_000683321542-if00 -> ../../ttyACM0
```

---

 ## 4\. Run the automated flash test

 The test script is located at:

```
/home/louis/flash_hello_test.sh
```

 Execute it with:

```
/home/louis/flash_hello_test.sh
```

 Alternatively:

```
~/flash_hello_test.sh
```

 The script automatically:

 1. Checks that the Zephyr firmware image exists.
2. Checks that `/dev/ttyACM0` exists.
3. Starts a Python serial listener.
4. Runs `west flash` using the J-Link runner.
5. Waits for the nRF52840 to boot.
6. Monitors `/dev/ttyACM0`.
7. Checks for the expected message.

 The expected UART output is similar to:

```
RECEIVED: b'*** Booting Zephyr OS build v4.5.0-rc1-43-ga90cd40a5f0a ***\r\nHello World! nrf52840dk/nrf52840\r\n'
```

 A successful test ends with:

```
========================================
  nRF52840 Hello World TEST: PASS
========================================

Flash succeeded and the expected UART
message was received.
```

 ## 5\. Interpretation

 A `PASS` means that the complete flash-and-boot path has been verified:

```
west flash
    ↓
J-Link
    ↓
nRF52840
    ↓
Zephyr firmware
    ↓
UART
    ↓
/dev/ttyACM0
    ↓
Python listener
```

 This provides a known-good baseline before development of the actual IoT PoC.

 If the test reports `FAIL`, investigate the hardware/connection/development environment before debugging the application itself.

 ## 6\. Recommended use

 Keep `flash_hello_test.sh` as a permanent smoke test for the PCB development environment.

 When starting work on a new application, run:

```
~/flash_hello_test.sh
```

 If it passes, the basic J-Link, flashing, firmware boot, and UART communication infrastructure has been verified.

 This would work well as a `README.md` section or as a dedicated file such as `docs/pcb-flash-smoke-test.md`.
