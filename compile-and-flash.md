## Step 1 — Windows PowerShell: check USB

powershell
```
usbipd list
```

## Step 2 — Windows PowerShell: attach J-Link to WSL

Replace `1-4` if the BUSID differs.

powershell
```
usbipd attach --wsl --busid 1-4
```

## Step 3 — WSL Ubuntu: activate Python virtual environment

bash
```
pyenv activate zephyr-3.12
```

## Step 4 — Verify development environment

bash
```
python --version
python -c "import sys; print(sys.executable)"
west --version
python -c "import serial; print(serial.__version__)"
```

## Step 5 — Verify USB and UART

bash
```
lsusb
ls -l /dev/ttyACM0
ls -l /dev/serial/by-id/
```

## Step 6 — Enter Zephyr workspace

bash
```
cd ~/zephyrproject
west topdir
```

## Step 7 — Compile the SSP Hello World firmware

bash
```
west build -p always \
  -b nrf52840dk/nrf52840 \
  -d /tmp/ssp-hello-world \
  ~/ssp-iot-poc/device/firmware/hello_world
```

## Step 8 — Verify compiled firmware

bash
```
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.hex
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.elf
```

## Step 9 — Flash the board

bash
```
west flash -d /tmp/ssp-hello-world -r jlink
```

## Step 10 — Run automated flash and UART smoke test

bash
```
~/flash_hello_test.sh
```

## Step 11 — Expected result

text
```
nRF52840 Hello World TEST: PASS
```
