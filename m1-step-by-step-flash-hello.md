# step by step flash of hello-world

## Step 1 — Windows PowerShell: check USB

powershell
```
usbipd list
```

## Step 2 — Windows PowerShell: attach J-Link to WSL

Replace `1-4` with the current BUSID if different.

powershell
```
usbipd attach --wsl --busid 1-4
```

## Step 3 — WSL Ubuntu: activate the Python virtual environment

bash
```
pyenv activate zephyr-3.12
```

## Step 4 — Verify Python environment

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

## Step 6 — Enter the Zephyr workspace

bash
```
cd ~/zephyrproject
west topdir
```

## Step 7 — Verify firmware images

bash
```
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.hex
ls -lh /tmp/ssp-hello-world/zephyr/zephyr.elf
```

## Step 8 — Flash the board

bash
```
west flash -d /tmp/ssp-hello-world -r jlink
```

## Step 9 — Run the complete flash and UART smoke test

bash
```
~/flash_hello_test.sh
```

## Step 10 — Confirm success

text
```
nRF52840 Hello World TEST: PASS
```
