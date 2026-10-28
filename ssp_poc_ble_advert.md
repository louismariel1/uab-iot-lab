# SSP BLE Firmware Flashing and UART Startup Validation

This procedure flashes the SmartSecure-Perimeter (SSP) BLE firmware to the nRF52840 development board and verifies that the firmware initializes Bluetooth and starts BLE advertising.

## Prerequisites

- Repository available at `~/ssp-iot-poc`
- Python environment `zephyr-3.12` activated through `pyenv`
- Zephyr workspace and `west` command available
- nRF52840 development board connected through J-Link
- UART serial device available at `/dev/ttyACM0`
- Existing firmware build artifacts in `/tmp/ssp-ble-peripheral`

## Procedure

Run the following commands from the WSL terminal:

```bash
cd ~/ssp-iot-poc

# Activate the Python environment containing west.
pyenv activate zephyr-3.12

# Verify west is available.
west --version

# Flash firmware and verify startup through UART.
./tools/test_ble_integration.sh
```

## Expected result

The environment check should report a valid `west` version, for example:

```text
West version: v1.5.0
```

The integration test flashes the existing firmware build using the J-Link runner and listens for startup messages over UART.

A successful run should include:

```text
Flashing SSP BLE firmware...
-- west flash: using runner jlink
JLink version: 9.82
Flashing file: /tmp/ssp-ble-peripheral/zephyr/zephyr.hex

UART: *** Booting Zephyr OS build v4.5.0-rc1-43-ga90cd40a5f0a ***
UART: SSP BLE peripheral starting
UART: SSP event subsystem initialized
UART: Bluetooth initialized
UART: SSP advertising started: SSP-POC

PASS: SSP BLE firmware started advertising.
```

## Validation status

**Result: PASS**

The recorded test confirmed that:

1. The firmware was flashed to the nRF52840 board.
2. Zephyr booted successfully.
3. The SSP event subsystem initialized.
4. Bluetooth initialized successfully.
5. The firmware reported that BLE advertising started with the device name `SSP-POC`.

The output `ninja: no work to do.` indicates that no rebuild was needed during this flash operation.

## Troubleshooting

If the test reports `pyenv: west: command not found`, activate the correct Python environment before rerunning the test:

```bash
pyenv activate zephyr-3.12
west --version
```

Proceed only after `west --version` succeeds.

## Next step: iPhone BLE connection test

After the startup integration test passes, run:

```bash
./tools/check_ble_connection.sh
```

Follow the script prompts and use **nRF Connect for Mobile** on the iPhone to scan for `SSP-POC` and attempt a connection.

**Scope of this validation:** the integration test verifies firmware startup and the advertising-start report over UART. It does not, by itself, verify successful iPhone connection, GATT service discovery, characteristic reads, or notifications. Those require separate tests.
