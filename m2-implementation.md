Your concern is justified. In IoT projects, it is easy to spend too much time perfecting a sensor abstraction, debating data types, or debugging individual modules before having anything visible to demonstrate.

For SSP, I would adopt a vertical-slice-first strategy: get the smallest complete Device → BLE → Android path working as early as possible, then improve the implementation incrementally.

There is one important correction to the milestone terminology:

- M2.7 is the step where we freeze the device-side sensor/data interface.
- Stage 7 in our previous plan is BLE implementation, which follows that interface definition.
- We do not need to complete every physical-sensor calibration task before writing BLE code. We can use synthetic events initially and replace them with physical measurements without changing the BLE contract.

So the goal is not to skip sound engineering. It is to avoid making every refinement a prerequisite for seeing the system work.

# 1. Proposed development strategy: reach BLE quickly

I recommend restructuring the work into a sequence of short, demonstrable increments.

### Step 0 — M1: environment

Done

Zephyr build environment, Git repository and Hello World flash test.

### Step 1 — Minimal SSP firmware

Define one compact event structure, sequence number, uptime timestamp and a deterministic synthetic motion event.

Deliverable: the firmware generates an event and prints it over UART.

### Step 2 — BLE peripheral on the nRF52840

Add advertising, one SSP GATT service, one event characteristic and notifications.

Deliverable: a phone can discover the board and receive events.

### Step 3 — Minimal Android BLE receiver

Scan, connect, discover the characteristic, subscribe to notifications and display decoded events.

Deliverable: a visible event stream in the Android app.

### Step 4 — Connect real motion detection

Replace synthetic X/Y/Z values with the physical IMU behind the same acquisition interface.

Deliverable: physical movement produces an event visible on Android.

### Step 5 — Stabilize and freeze interfaces

Validate packet decoding, sequence handling, reconnection and repeatability. Document the interfaces for the next milestones.

Deliverable: the Device → Edge vertical slice is reproducible from Git.

This is a deliberate adjustment to our earlier M2 sequence. We will still test the physical IMU and calibrate motion detection, but we can develop and demonstrate BLE in parallel with that work.

The first visible end-to-end milestone should be synthetic event → BLE notification → Android screen. That proves the communication architecture while keeping sensor-driver problems out of the BLE debugging process.

# 2. Script-by-script development plan

I suggest that we do not write all the firmware from scratch in one large operation. Instead, we use a small number of scripts, each with one testable outcome.

The main optimization is to reuse Zephyr's working BLE implementation first, then add SSP functionality to it. Zephyr provides Bluetooth peripheral samples that demonstrate advertising, GATT services and BLE operation. GitHub

+1

## Step 1 — Verify that BLE works on the board

Before developing our own BLE service, test a stock Zephyr sample.

From the project environment:

bash
```
export ZEPHYR_BASE=~/zephyrproject/zephyr

ls "$ZEPHYR_BASE/samples/bluetooth"
```

If `peripheral_nus` is present, use it for a simple text-based BLE connection. Otherwise, start with the generic `peripheral` sample and inspect its available characteristics. The Nordic UART Service sample is documented as a simple BLE echo server. Zephyr Project Documentation

+1

For example, if `peripheral_nus` exists:

bash
```
west build -p always \
  -d /tmp/ssp-ble-probe \
  -b nrf52840dk/nrf52840 \
  "$ZEPHYR_BASE/samples/bluetooth/peripheral_nus"
```

Then:

bash
```
west flash -d /tmp/ssp-ble-probe -r jlink
```

Open the Android app nRF Connect for Mobile, scan for the device, connect, and inspect the available GATT service and characteristics.

- If this works, we have verified the BLE hardware and basic stack independently of SSP.
- If it fails, we debug the sample environment rather than our application code.

Do not change the existing Hello World application or its verification script.

## Step 2 — Create a separate SSP BLE application

Once the BLE probe works, create a dedicated application:

text
```
device/firmware/
├── hello_world/              # Preserve M1
├── ssp_ble_demo/             # New working application
│   ├── CMakeLists.txt
│   ├── prj.conf
│   └── src/
│       ├── main.c
│       ├── ssp/
│       │   ├── ssp_event.h
│       │   └── ssp_motion.c
│       ├── sensor/
│       │   ├── sensor_interface.h
│       │   └── sensor_sim.c
│       └── ble/
│           ├── ssp_ble.h
│           └── ssp_ble.c
```

The purpose of each file is deliberately narrow.

| File | Responsibility |
| --- | --- |
| `main.c` | Initialize modules and run the main loop |
| `ssp_event.h` | Define the logical event and measurement representation |
| `ssp_motion.c` | Calculate the motion metric and classify movement |
| `sensor_interface.h` | Provide a sensor-independent acquisition interface |
| `sensor_sim.c` | Generate repeatable synthetic measurements |
| `ssp_ble.h` | Expose the BLE initialization and event-sending functions |
| `ssp_ble.c` | Implement the GATT service, advertising and notifications |

This is enough modularity to prevent everything from ending up in `main.c`, without introducing an elaborate architecture.

## Step 3 — Define the smallest useful SSP event

Do not start by designing a comprehensive protocol. Start with the fields necessary for the demonstration.

For example:

c
```
enum ssp_event_type {
    SSP_EVENT_NONE = 0,
    SSP_EVENT_MOTION = 1
};

struct ssp_event {
    uint16_t sequence;
    uint32_t timestamp_ms;
    enum ssp_event_type type;
};
```

The application will eventually include acceleration and motion metrics in its internal measurement structure. They do not all need to be sent over BLE in the first demonstration.

For the initial BLE message, we can use a short text representation such as:

text
```
SSP,1,42,123456
```

Here, the fields are protocol identifier, event type, sequence number and device uptime in milliseconds.

This is an initial proposal, not the final protocol specification. The actual BLE payload encoding and UUIDs must be documented and frozen before the BLE interface is treated as stable.

## Step 4 — Implement synthetic events

The synthetic sensor should produce a repeatable sequence:

text
```
NONE
NONE
NONE
MOTION_EVENT
NONE
NONE
NONE
MOTION_EVENT
...
```

The important point is that `main.c` must call the same event-processing and BLE-sending functions that the physical IMU will eventually use.

For example, the application flow should be:

c
```
sensor_read(&sample);

event = ssp_process_sample(&sample);

if (event.type == SSP_EVENT_MOTION) {
    ssp_ble_send_event(&event);
}
```

This is conceptual code showing the intended interface. We will align the actual function signatures when implementing the files.

Acceptance test: UART shows the expected sequence of synthetic events, and no BLE functionality has been mixed into the motion-detection algorithm.

## Step 5 — Add the BLE service

This is the key implementation step.

`ssp_ble.c` should:

1. Initialize the Bluetooth stack.
2. Register the SSP GATT service.
3. Start connectable advertising.
4. Expose an event characteristic with notification support.
5. Track whether a client has enabled notifications.
6. Send event data when a notification subscriber is connected.

Zephyr's GATT APIs provide the mechanisms for service registration and characteristic notifications. Zephyr Project Documentation

+1

For the first implementation, we should avoid unnecessary features such as bonding, encryption policy customization, multiple-device management and dynamic sampling configuration. We can add them after the first successful connection.

Acceptance test: the Android phone discovers the SSP service and receives a notification when the firmware generates a synthetic motion event.

## Step 6 — Validate the BLE protocol before building a full Android app

Use nRF Connect for Mobile as a temporary diagnostic tool.

This gives us an inexpensive way to test:

- Advertising and device discovery.
- Connection and GATT service discovery.
- Characteristic properties.
- Notification subscription.
- Actual payload bytes or text.
- Repeated events.

Record a screenshot and a short UART log. These are already useful project evidence.

This is also where we catch protocol mismatches before writing a dedicated Android application.

## Step 7 — Build the minimal Android Edge application

Once the firmware produces stable BLE notifications, create a minimal Kotlin Android application.

I would use only four classes initially:

| Class | Responsibility |
| --- | --- |
| `BleScanner` | Scan for the SSP device |
| `BleConnectionManager` | Connect, discover services and manage notification subscriptions |
| `SspEventParser` | Validate and decode the event payload |
| `MainActivity` | Display connection status, incoming events and the SSP state |

The UI needs only:

- Connection status.
- Latest sequence number and timestamp.
- Latest received event.
- Current SSP state: `NORMAL` or `ELEVATED`.

The initial state transition can be simple:

text
```
NORMAL
   |
   | valid MOTION_EVENT
   v
ELEVATED
```

We can then add disconnect handling and recovery, followed by integration with the real IMU.

Acceptance test: a synthetic event generated by the nRF52840 appears in the Android app and triggers the expected state transition.

At that point, we have reached the useful version of Stage 7: BLE works, the Edge receives the event, and the product's core behavior is visible.

# 3. Can code generation automate this process?

Yes. There are several useful approaches, but I would not introduce a large model-driven engineering framework just to generate a few C files and Kotlin classes.

The best solution for SSP is a combination of working templates, a small protocol definition, automated builds and tests, and AI-assisted implementation.

Why We Moved from FreeRTOS to Zephyr

### Option A — Zephyr samples and application templates

Start here

Reuse the existing BLE stack and sample code, then customize the GATT service and event logic. This is the fastest route to a functioning device.

Best for: reducing low-level BLE implementation and debugging time.

[Zephyr Bluetooth samples](<https://docs.zephyrproject.org/latest/samples/bluetooth/bluetooth.html>)

Google Protocol Buffers: A Game Changer for IPC Mechanisms | by Nikheel Vishwas Savant | Medium

### Option B — Schema-driven data generation

Define the event schema once and generate serialization or data-access code where supported. Protocol Buffers with Nanopb is one possibility for embedded C; a small manually specified binary packet may be simpler for our first BLE event.

Best for: keeping device and Edge payload definitions consistent as the protocol grows.

[Nanopb documentation and repository](<https://github.com/nanopb/nanopb>)

precis0x (@precisox) on X

### Option C — Automated build and validation

Use `west build`, test scripts and CI to catch compilation errors and interface mismatches immediately. Generated code is only useful if it can be built and tested automatically.

Best for: avoiding repetitive debugging loops and preventing regressions.

[Zephyr Twister testing framework](<https://docs.zephyrproject.org/latest/develop/test/twister.html>)

Jual Github Copilot 12 Months Private | Shopee Indonesia

### Option D — AI-assisted code generation

Generate one module at a time from explicit requirements, with the existing Zephyr version and board configuration supplied as context. Have the tool build the application and fix compiler errors before moving to the next module.

Best for: rapidly producing C and Kotlin scaffolding, tests and documentation.

## What about a DSL?

A domain-specific language could describe the system in a compact form:

yaml
```
device:
  id: SSP-001
  platform: nrf52840dk
  rtos: zephyr

sensor:
  source: simulated
  fields: [accel_x, accel_y, accel_z]

motion:
  metric: acceleration_magnitude
  event: MOTION_EVENT

ble:
  role: peripheral
  service: ssp_service
  characteristic: event
  properties: [read, notify]

edge:
  platform: android
  language: kotlin
  on_event: update_ssp_state
```

A generator could use this file to produce:

- C event definitions.
- Protocol constants and UUIDs.
- Kotlin event definitions and parser scaffolding.
- Test vectors.
- Documentation tables.

However, a DSL alone does not generate a correct working IoT system. It still needs templates or generators for Zephyr, GATT and Android, plus tests of the generated output. A custom generator is worthwhile only if we expect the schema to evolve often or need to produce several similar devices.

For SSP, I would start with a YAML protocol definition and a few small generation scripts, not a new framework that becomes another project to maintain.

# 4\. My recommended automation pipeline

SSP specification

YAML event model + GATT UUIDs + payload format

Code generation / templates

C headers · packet encoder · Kotlin parser · test vectors

Automated validation

Firmware build · host-side unit tests · Android build

Hardware verification

Flash · connect · receive notification · inspect UART and Android logs

The hardware verification stage remains necessary. A successful compile does not prove that BLE advertising, connections or notifications work on the actual board.

## 5\. The rules that will keep us moving

I would enforce five development rules throughout M2 and the subsequent BLE work:

1. One working increment at a time. Every step must build or demonstrate something before we add another feature.
2. No speculative complexity. Do not introduce GNSS, ML, advanced filtering or configurable policies before the basic path works.
3. Keep the first protocol small. Freeze the minimum event fields and UUIDs early; evolve them deliberately.
4. Build automatically after each module. Avoid accumulating changes from several unverified files.
5. Keep a known-good checkpoint. Commit each working increment and retain the original Hello World application.

My preferred route is to verify the existing Zephyr BLE sample, create `ssp_ble_demo`, implement synthetic SSP events and notifications, and then build the minimal Android receiver. That gives us visible progress quickly without abandoning the M2 acceptance criteria or creating unnecessary tooling.
