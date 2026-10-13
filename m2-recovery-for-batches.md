## 1. First: secure the recovered baseline

Run these commands from `/home/louis/ssp-iot-poc`. They do not modify the firmware.

bash
```
cd ~/ssp-iot-poc

# Inspect the current firmware before committing it
cat device/firmware/hello_world/CMakeLists.txt
cat device/firmware/hello_world/prj.conf
cat device/firmware/hello_world/src/main.c

# Check for any additional files and accidental secrets
find device -type f -print
git status --short
```

If the files contain the expected project code and no credentials or other sensitive information, commit the recovered device baseline:

bash
```
git add device/
git diff --cached --stat
git diff --cached
git commit -m "Recover and checkpoint SSP device firmware baseline"
```

This creates a recoverable checkpoint without changing the existing application. Review the staged diff before committing.

## 2. Run a focused environment audit

Next, execute this block. It gathers the information needed to generate code compatible with your actual environment.

bash
```

cd ~/ssp-iot-poc

echo "===== GIT ====="
git status --short
git log -2 --oneline

echo "===== PYTHON AND WEST ====="
pyenv activate zephyr-3.12
python --version
west --version
echo "ZEPHYR_BASE=$ZEPHYR_BASE"

echo "===== ZEPHYR WORKSPACE ====="
cd ~/zephyrproject
west topdir
west list

echo "===== BLUETOOTH SAMPLES ====="
find "$ZEPHYR_BASE/samples/bluetooth" \
  -maxdepth 2 -name CMakeLists.txt -print

echo "===== SSP REPOSITORY FILES ====="
cd ~/ssp-iot-poc
find device edge docs -type f -print

echo "===== EXISTING TEST SCRIPT ====="
if [ -f "$HOME/flash_hello_test.sh" ]; then
    sed -n '1,240p' "$HOME/flash_hello_test.sh"
else
    echo "Flash test script not found"
fi
```

If `ZEPHYR_BASE` is empty, stop at that point and establish the correct workspace path before running the Bluetooth sample discovery command. The earlier instructions used `~/zephyrproject/zephyr` as the expected path.

Please paste the output here. It will tell us which Zephyr APIs and Bluetooth samples are actually available, and whether the current smoke test assumes a particular UART message or board state.

## 3. Then generate code in controlled batches

Once we have that audit, this is the workflow I recommend.

Batch A — Contract and test specification

Define the initial event payload, UUIDs, characteristic properties, packet validation rules and shared test vectors. Establish which files are generated and which are preserved.

Batch B — Complete Zephyr firmware

Generate the synthetic sensor, SSP event model, motion-processing pipeline, BLE peripheral, notification handling, configuration and automated tests as one coherent patch.

Batch C — Complete Android Edge application

Generate the Kotlin BLE receiver, event parser, validation, state machine, minimal UI and unit tests against the same protocol contract.

Batch D — Automated integration and repair

Build both components, run tests, repair failures using exact logs, review the final diff and checkpoint each verified increment in Git.

### Why I would not generate all the code immediately

The firmware is only a few files today, and the Android directory may be empty. We do not yet know the installed Zephyr version, available BLE sample, Android SDK configuration or precise build interfaces.

Generating everything before checking these details risks producing a large batch that fails for several unrelated reasons. A short audit now should save considerably more time during integration.

Our next concrete objective is a synthetic SSP event received by Android over BLE, followed by the `NORMAL → ELEVATED` transition. We will then substitute the real IMU for the simulator and complete the physical movement test.
