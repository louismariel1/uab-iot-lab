# 1\. Tasks we can do now

 ### M2 preparation — possible without sensors

 1. **Create the SSP device firmware structure**
   - Separate application logic from sensor drivers.
   - Establish directories for sensors, data model, motion processing and events.
2. **Define the SSP device data model**
   - Device ID
   - Sequence
   - Timestamp
   - X/Y/Z acceleration
   - Motion metric
   - Event type
   - Status
3. **Define SSP event types**
   - `NONE`
   - `MOTION_EVENT`
4. **Implement measurement sequencing**
   - Increment sequence number for every measurement.
5. **Implement timestamp handling**
   - Use Zephyr uptime initially.
   - Later decide whether GNSS time should become the authoritative timestamp.
6. **Implement the motion metric**
   - Calculate acceleration magnitude from X/Y/Z.
7. **Implement the deterministic motion detector**
   - Configurable threshold.
   - Generate `MOTION_EVENT`.
8. **Implement a sensor-independent acquisition interface**
   - The motion-processing code should not care whether the data came from a Bosch, ST, etc. IMU.
9. **Implement UART diagnostic output**
   - Make measurements/events visible.
10. **Implement synthetic sensor data**
    - Feed artificial X/Y/Z values into the exact same pipeline.
    - This allows us to test the firmware architecture before the physical IMU arrives.
11. **Build and flash this firmware**
    - Verify it on the nRF52840-DK.
12. **Perform a software-level M2 test**
    - Stationary sample → `NONE`
    - Movement sample → `MOTION_EVENT`

 ### Things we should _not_ implement yet

 Until we know the exact hardware:

 - IMU-specific driver configuration
- IMU GPIO/I²C/SPI pin assignments
- GNSS driver configuration
- GNSS UART pin assignments
- GNSS parsing
- Actual sensor calibration
- BLE packet encoding

 Those should be based on the actual modules.

---

 # 2\. Proposed implementation structure

 I'd change our current tiny application from:

```
hello_world/
├── CMakeLists.txt
├── prj.conf
└── src/
    └── main.c
```

 to:

```
hello_world/
├── CMakeLists.txt
├── prj.conf
└── src/
    ├── main.c
    │
    ├── ssp/
    │   ├── ssp_types.h
    │   ├── ssp_motion.h
    │   └── ssp_motion.c
    │
    └── sensor/
        ├── sensor_interface.h
        └── sensor_sim.c
```

 Eventually we'll have:

```
sensor/
├── sensor_interface.h
├── imu.c
└── gnss.c
```

 The important architectural boundary is:

```
                SENSOR HARDWARE
                      │
             ┌────────┴────────┐
             │                 │
           IMU driver       GNSS driver
             │                 │
             └────────┬────────┘
                      ▼
               SSP data model
                      │
               local processing
                      │
                 SSP events
                      │
                  BLE later
```

 That means tomorrow's physical IMU can replace our simulator **without rewriting the motion-processing logic**.

---

 # 3\. Implementation Step 1 — create the structure

 From:

```
(zephyr-3.12) louis@LMDesktop:~/ssp-iot-poc$
```

 run:

```
mkdir -p device/firmware/hello_world/src/ssp
mkdir -p device/firmware/hello_world/src/sensor
```

 Then verify:

```
find device/firmware/hello_world -maxdepth 3 -type d -print
```

 You should see:

```
device/firmware/hello_world
device/firmware/hello_world/src
device/firmware/hello_world/src/ssp
device/firmware/hello_world/src/sensor
```

---

 # 4\. Implementation Step 2 — define the SSP data model

 Create:

```
cat > device/firmware/hello_world/src/ssp/ssp_types.h <<'EOF'
#ifndef SSP_TYPES_H
#define SSP_TYPES_H

#include <stdint.h>

#define SSP_DEVICE_ID "SSP-001"

enum ssp_event_type {
    SSP_EVENT_NONE = 0,
    SSP_EVENT_MOTION = 1
};

enum ssp_status {
    SSP_STATUS_OK = 0,
    SSP_STATUS_SENSOR_ERROR = 1
};

struct ssp_motion_measurement {
    uint32_t timestamp_ms;
    uint32_t sequence;

    float acceleration_x;
    float acceleration_y;
    float acceleration_z;

    float motion_metric;

    enum ssp_event_type event;
    enum ssp_status status;
};

#endif /* SSP_TYPES_H */
EOF
```

 This is our first real SSP software abstraction.

 Notice that nothing here knows whether the sensor is:

 - Bosch
- ST
- Analog Devices
- another manufacturer

 That's intentional.

---

 # 5\. Implementation Step 3 — create the motion-processing interface

 Create:

```
cat > device/firmware/hello_world/src/ssp/ssp_motion.h <<'EOF'
#ifndef SSP_MOTION_H
#define SSP_MOTION_H

#include "ssp_types.h"

float ssp_calculate_motion_metric(float x, float y, float z);

enum ssp_event_type ssp_detect_motion(float motion_metric);

#endif /* SSP_MOTION_H */
EOF
```

 Now implement it:

```
cat > device/firmware/hello_world/src/ssp/ssp_motion.c <<'EOF'
#include <math.h>

#include "ssp_motion.h"

/*
 * Initial experimental threshold.
 *
 * This is deliberately not considered a final SSP value.
 * It will be calibrated using the real IMU during M2.
 */
#define SSP_MOTION_THRESHOLD 1.20f

float ssp_calculate_motion_metric(float x, float y, float z)
{
    return sqrtf((x * x) + (y * y) + (z * z));
}

enum ssp_event_type ssp_detect_motion(float motion_metric)
{
    if (motion_metric > SSP_MOTION_THRESHOLD) {
        return SSP_EVENT_MOTION;
    }

    return SSP_EVENT_NONE;
}
EOF
```

 ### Why start with this?

 Because now the core motion logic is testable independently of the physical sensor.

 Tomorrow the real IMU gives us:

```
x
y
z
```

 and we simply pass those values into:

```
ssp_calculate_motion_metric(x, y, z);
```

---

 # 6\. Implementation Step 4 — define the sensor abstraction

 Create:

```
cat > device/firmware/hello_world/src/sensor/sensor_interface.h <<'EOF'
#ifndef SSP_SENSOR_INTERFACE_H
#define SSP_SENSOR_INTERFACE_H

struct sensor_sample {
    float acceleration_x;
    float acceleration_y;
    float acceleration_z;
};

int sensor_init(void);

int sensor_read(struct sensor_sample *sample);

#endif /* SSP_SENSOR_INTERFACE_H */
EOF
```

 This is important.

 Our application will eventually say:

```
sensor_read(&sample);
```

 It doesn't need to know whether that means:

```
I²C → IMU register → conversion
```

 or:

```
SPI → IMU register → conversion
```

 or something else.

---

 # 7\. Implementation Step 5 — create a simulated sensor

 Until tomorrow, we'll use synthetic data.

 Create:

```
cat > device/firmware/hello_world/src/sensor/sensor_sim.c <<'EOF'
#include "sensor_interface.h"

#include <zephyr/kernel.h>

/*
 * Synthetic sensor for software validation.
 *
 * The real IMU implementation will replace this module
 * once the laboratory sensor is identified.
 */

static uint32_t sample_count;

int sensor_init(void)
{
    sample_count = 0;
    return 0;
}

int sensor_read(struct sensor_sample *sample)
{
    if (sample == NULL) {
        return -1;
    }

    sample_count++;

    /*
     * First five samples simulate a stationary device.
     * The next sample simulates movement.
     * The sequence then repeats.
     */

    if ((sample_count % 10) == 0) {
        sample->acceleration_x = 1.80f;
        sample->acceleration_y = 0.40f;
        sample->acceleration_z = 1.50f;
    } else {
        /*
         * Approximate 1 g on the Z axis while stationary.
         */
        sample->acceleration_x = 0.02f;
        sample->acceleration_y = -0.01f;
        sample->acceleration_z = 1.00f;
    }

    return 0;
}
EOF
```

 This gives us a controlled test:

```
samples 1–9
    ↓
stationary
    ↓
NONE

sample 10
    ↓
simulated movement
    ↓
MOTION_EVENT
```

---

 # 8\. Implementation Step 6 — replace main.c

 Now we connect everything.

```
cat > device/firmware/hello_world/src/main.c <<'EOF'
#include <zephyr/kernel.h>
#include <zephyr/sys/printk.h>

#include "ssp/ssp_types.h"
#include "ssp/ssp_motion.h"
#include "sensor/sensor_interface.h"

#define SSP_SAMPLE_INTERVAL_MS 500

int main(void)
{
    struct sensor_sample sample;
    struct ssp_motion_measurement measurement;

    uint32_t sequence = 0;

    printk("\n");
    printk("========================================\n");
    printk("  SSP Device Firmware\n");
    printk("  M2 Motion Acquisition Test\n");
    printk("========================================\n");
    printk("Device ID: %s\n", SSP_DEVICE_ID);
    printk("\n");

    if (sensor_init() != 0) {
        printk("ERROR: sensor initialization failed\n");
        return 0;
    }

    printk("Sensor initialized\n");
    printk("Starting acquisition...\n\n");

    while (1) {

        if (sensor_read(&sample) != 0) {
            printk("ERROR: sensor read failed\n");
            k_msleep(SSP_SAMPLE_INTERVAL_MS);
            continue;
        }

        measurement.timestamp_ms = k_uptime_get_32();
        measurement.sequence = sequence++;

        measurement.acceleration_x = sample.acceleration_x;
        measurement.acceleration_y = sample.acceleration_y;
        measurement.acceleration_z = sample.acceleration_z;

        measurement.motion_metric =
            ssp_calculate_motion_metric(
                sample.acceleration_x,
                sample.acceleration_y,
                sample.acceleration_z
            );

        measurement.event =
            ssp_detect_motion(measurement.motion_metric);

        measurement.status = SSP_STATUS_OK;

        printk(
            "seq=%u time=%u "
            "accel=(%.2f, %.2f, %.2f) "
            "motion=%.2f "
            "event=%s\n",
            measurement.sequence,
            measurement.timestamp_ms,
            (double)measurement.acceleration_x,
            (double)measurement.acceleration_y,
            (double)measurement.acceleration_z,
            (double)measurement.motion_metric,
            measurement.event == SSP_EVENT_MOTION
                ? "MOTION_EVENT"
                : "NONE"
        );

        k_msleep(SSP_SAMPLE_INTERVAL_MS);
    }

    return 0;
}
EOF
```

---

 # 9\. Implementation Step 7 — update CMakeLists.txt

 Our current CMake file only compiles `main.c`.

 We need to add our new modules.

 Replace it with:

```
cat > device/firmware/hello_world/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(ssp_hello_world)

target_sources(app PRIVATE
    src/main.c
    src/ssp/ssp_motion.c
    src/sensor/sensor_sim.c
)
EOF
```

---

 # 10\. Implementation Step 8 — update configuration

 Our current `prj.conf` is enough for the basic firmware, but let's explicitly retain the required console functionality:

```
cat > device/firmware/hello_world/prj.conf <<'EOF'
CONFIG_PRINTK=y
CONFIG_SERIAL=y
CONFIG_CONSOLE=y
CONFIG_UART_CONSOLE=y
EOF
```

 No BLE yet.

 No IMU driver yet.

 No GNSS driver yet.

 That's deliberate.

---

 # 11\. Implementation Step 9 — compile it

 Now we can test the entire software architecture **without the physical sensor**:

```
export ZEPHYR_BASE=~/zephyrproject/zephyr

west build -p always \
  -d /tmp/ssp-m2-test \
  -b nrf52840dk/nrf52840 \
  device/firmware/hello_world
```

 The important result is that the application still produces:

```
zephyr.elf
zephyr.hex
```

 and the build succeeds.

---

 # 12\. Implementation Step 10 — flash and validate

 Once the build succeeds, our existing verification script can be adapted later.

 For now, because the expected output is no longer:

```
Hello World! nrf52840dk/nrf52840
```

 we should **not modify the established test script yet**.

 Instead, we can flash manually:

```
west flash -d /tmp/ssp-m2-test -r jlink
```

 Then connect to `/dev/ttyACM0` and observe the output.

 We expect something conceptually like:

```
========================================
  SSP Device Firmware
  M2 Motion Acquisition Test
========================================
Device ID: SSP-001

Sensor initialized
Starting acquisition...

seq=0 time=1234 accel=(0.02, -0.01, 1.00) motion=1.00 event=NONE
seq=1 time=1734 accel=(0.02, -0.01, 1.00) motion=1.00 event=NONE
...
seq=9 time=5734 accel=(1.80, 0.40, 1.50) motion=2.36 event=MOTION_EVENT
...
```

 That gives us our first complete **software-level M2 pipeline**:

```
Synthetic sensor
      ↓
Sensor abstraction
      ↓
SSP measurement
      ↓
Timestamp + sequence
      ↓
Motion metric
      ↓
Motion detector
      ↓
MOTION_EVENT
      ↓
UART
```

 ## What tomorrow's IMU changes

 Very little of the architecture.

 Currently:

```
sensor_sim.c
      ↓
synthetic X/Y/Z
```

 Tomorrow:

```
imu.c
      ↓
actual X/Y/Z
```

 The rest remains:

```
actual IMU
     ↓
sensor interface
     ↓
SSP measurement
     ↓
motion metric
     ↓
MOTION_EVENT
```

 That is exactly the separation we want.

 ### And GNSS comes afterward

 GNSS will eventually become another independent acquisition module:

```
                 ┌── IMU ──→ motion
                 │
nRF52840 ────────┤
                 │
                 └── GNSS ─→ position
```

 We should **not write GNSS-specific code now**, because we don't yet know which module the lab is giving us. Once you have the exact IMU and GNSS model numbers, we can replace the simulator with the real Zephyr sensor implementation and add GNSS without redesigning the SSP core.

 **I recommend we execute Steps 1–8 now, build in Step 9, and paste the build output here before flashing.** That gives us a clean checkpoint before we introduce the real hardware.
