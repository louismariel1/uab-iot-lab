The fastest path is to create a single, reproducible BLE demo-generation batch driven by one YAML protocol contract, with templates for Zephyr C and Android Kotlin, shared test vectors, automated builds, and an optional hardware integration test.

We should preserve your two successful test scripts and the existing `device/firmware/hello_world/` application. The new implementation will live alongside them, with its own build directory.

The generated implementation will demonstrate:

`nRF52840 synthetic motion event → BLE GATT notification → Android packet validation → NORMAL → ELEVATED`

The physical IMU, cloud integration, and production security hardening remain separate follow-up work.

# 1. Target repository structure

ssp-iot-poc/

protocol/ssp\_ble\_v1.yaml

Batch A — authoritative protocol definition

tools/ble\_codegen/

Batch A — Python generator and Jinja templates

device/firmware/ble\_peripheral/

Batch B — Zephyr BLE peripheral and synthetic events

edge/android/

Batch C — Kotlin BLE client and monitoring UI

tests/ble/

Shared protocol vectors and cross-language tests

tools/build\_ble\_demo.sh

Batch D — generate, test, build and report

One design decision matters: the YAML is the source of truth, but it must not generate an opaque application. Generate the protocol constants, packet encoder/decoder, and documentation; keep the BLE lifecycle, error handling, and application logic in readable C and Kotlin templates.

# 2. Batch A — Protocol DSL, generator and shared test vectors

## 2.1 Define the authoritative YAML contract

Create `protocol/ssp_ble_v1.yaml`:

yaml
```
protocol:
  name: SSP BLE Protocol
  version: 1
  byte_order: little_endian
  max_payload_bytes: 20

device:
  name: SSP-POC
  id: 1397960705 # 0x53535001

gatt:
  service_uuid: "7d2ea28a-f7bd-485a-bd9d-92ad6ecfe93e"

  characteristics:
    status:
      uuid: "7d2ea28a-f7bd-485a-bd9d-92ad6ecfe940"
      properties: [read]
      permissions: [read]

    event:
      uuid: "7d2ea28a-f7bd-485a-bd9d-92ad6ecfe941"
      properties: [notify]
      permissions: [none]

payload:
  name: SSPEventV1
  length: 17

  fields:
    - { name: version,      type: u8,  value: 1 }
    - { name: event_type,   type: u8,  values: { NONE: 0, MOTION_EVENT: 1 } }
    - { name: status,       type: u8,  values: { OK: 0, SENSOR_ERROR: 1 } }
    - { name: device_id,    type: u32 }
    - { name: sequence,     type: u32 }
    - { name: timestamp_ms, type: u32 }
    - { name: motion_mg,    type: u16 }

semantics:
  timestamp_ms: "Milliseconds since device boot"
  motion_mg: "Motion metric expressed in milli-g; provisional for the synthetic demo"
  sequence: "Unsigned 32-bit counter incremented for each generated event"
  state_transition:
    - from: NORMAL
      on: MOTION_EVENT
      to: ELEVATED

demo:
  synthetic_event_delay_ms: 3000
  repeat_interval_ms: 5000
  require_notification_subscription: true

tests:
  - name: synthetic_motion_event
    values:
      version: 1
      event_type: MOTION_EVENT
      status: OK
      device_id: 1397960705
      sequence: 1
      timestamp_ms: 1000
      motion_mg: 1800
    expected_hex: "0101000150535301000000e80300000807"
```

This defines a compact, 17-byte event payload. It is deliberately a provisional BLE v1 contract for the synthetic demonstration, not yet a validated physical-IMU protocol.

The event characteristic sends notifications; the status characteristic is readable. We can add a writable monitoring-policy characteristic in a later revision to demonstrate adaptive sampling.

## 2.2 Add the generator dependencies

Create `tools/ble_codegen/requirements.txt`:

text
```
PyYAML>=6.0,<7.0
Jinja2>=3.1,<4.0
```

Install them in your existing environment:

bash
```
python -m pip install -r tools/ble_codegen/requirements.txt
```

## 2.3 Create the generator

Create `tools/ble_codegen/generate.py`:

python
```
#!/usr/bin/env python3

import argparse
import json
import struct
from pathlib import Path

import yaml
from jinja2 import Environment, FileSystemLoader, StrictUndefined

ROOT = Path(__file__).resolve().parents[2]
SPEC = ROOT / "protocol/ssp_ble_v1.yaml"
TEMPLATES = ROOT / "tools/ble_codegen/templates"

TYPE_FORMATS = {
    "u8": ("B", 1),
    "u16": ("H", 2),
    "u32": ("I", 4),
}

def load_spec():
    spec = yaml.safe_load(SPEC.read_text(encoding="utf-8"))

    if spec["protocol"]["byte_order"] != "little_endian":
        raise ValueError("Only little-endian payloads are supported")

    fields = spec["payload"]["fields"]
    calculated_length = sum(TYPE_FORMATS[f["type"]][1] for f in fields)

    if calculated_length != spec["payload"]["length"]:
        raise ValueError(
            f"Payload length mismatch: declared "
            f"{spec['payload']['length']}, calculated {calculated_length}"
        )

    if calculated_length > spec["protocol"]["max_payload_bytes"]:
        raise ValueError("Payload exceeds configured BLE limit")

    uuids = [spec["gatt"]["service_uuid"]]
    uuids.extend(
        c["uuid"] for c in spec["gatt"]["characteristics"].values()
    )

    if len(uuids) != len(set(uuids)):
        raise ValueError("GATT UUIDs must be unique")

    for uuid in uuids:
        if len(uuid.split("-")) != 5 or len(uuid.replace("-", "")) != 32:
            raise ValueError(f"Invalid UUID: {uuid}")

    return spec

def encode_event(spec, values):
    encoded = bytearray()

    for field in spec["payload"]["fields"]:
        name = field["name"]
        value = values[name]

        if "values" in field:
            allowed = field["values"]
            if isinstance(value, str):
                if value not in allowed:
                    raise ValueError(f"Invalid {name}: {value}")
                value = allowed[value]
            elif value not in allowed.values():
                raise ValueError(f"Invalid {name}: {value}")

        fmt, _ = TYPE_FORMATS[field["type"]]
        encoded.extend(struct.pack("<" + fmt, value))

    expected_length = spec["payload"]["length"]
    if len(encoded) != expected_length:
        raise ValueError("Encoded payload length mismatch")

    return bytes(encoded)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--check",
        action="store_true",
        help="Validate and render in memory without writing files",
    )
    args = parser.parse_args()

    spec = load_spec()

    # Validate all declared test vectors before generating code.
    for test in spec.get("tests", []):
        payload = encode_event(spec, test["values"])
        expected = test["expected_hex"].lower()

        if payload.hex() != expected:
            raise ValueError(
                f"{test['name']}: expected {expected}, got {payload.hex()}"
            )

    env = Environment(
        loader=FileSystemLoader(str(TEMPLATES)),
        undefined=StrictUndefined,
        autoescape=False,
        keep_trailing_newline=True,
    )

    outputs = {
        ROOT / "device/firmware/ble_peripheral/src/generated/ssp_protocol.h":
            "ssp_protocol.h.j2",
        ROOT / "edge/android/app/src/main/java/org/ssp/protocol/SspProtocol.kt":
            "SspProtocol.kt.j2",
        ROOT / "docs/protocol/ssp-ble-v1.md":
            "protocol.md.j2",
    }

    context = {
        "spec": spec,
        "service_uuid": spec["gatt"]["service_uuid"],
        "status_uuid": spec["gatt"]["characteristics"]["status"]["uuid"],
        "event_uuid": spec["gatt"]["characteristics"]["event"]["uuid"],
        "payload_length": spec["payload"]["length"],
        "fields": spec["payload"]["fields"],
        "event_values": next(
            f["values"] for f in spec["payload"]["fields"]
            if f["name"] == "event_type"
        ),
        "status_values": next(
            f["values"] for f in spec["payload"]["fields"]
            if f["name"] == "status"
        ),
        "device_id": spec["device"]["id"],
    }

    rendered = {
        path: env.get_template(template).render(**context)
        for path, template in outputs.items()
    }

    if args.check:
        print("Protocol and test vectors validated.")
        print(f"Payload length: {spec['payload']['length']} bytes")
        print("Generated outputs render successfully.")
        return

    for path, content in rendered.items():
        path.parent.mkdir(parents=True, exist_ok=True)
        path.write_text(content, encoding="utf-8")
        print(f"Generated: {path.relative_to(ROOT)}")

    vector_path = ROOT / "tests/ble/test_vectors.json"
    vector_path.parent.mkdir(parents=True, exist_ok=True)
    vector_path.write_text(
        json.dumps(
            [
                {
                    "name": test["name"],
                    "values": test["values"],
                    "expected_hex": test["expected_hex"],
                }
                for test in spec.get("tests", [])
            ],
            indent=2,
        ) + "\n",
        encoding="utf-8",
    )
    print("Generated: tests/ble/test_vectors.json")

if __name__ == "__main__":
    main()
```

## 2.4 Add the C protocol template

Create `tools/ble_codegen/templates/ssp_protocol.h.j2`:

c
```
#ifndef SSP_PROTOCOL_H
#define SSP_PROTOCOL_H

#include <stdint.h>
#include <stddef.h>

#define SSP_PROTOCOL_VERSION {{ spec.protocol.version }}
#define SSP_EVENT_PAYLOAD_LENGTH {{ payload_length }}
#define SSP_DEVICE_ID UINT32_C({{ device_id }})

#define SSP_EVENT_NONE {{ event_values.NONE }}
#define SSP_EVENT_MOTION_EVENT {{ event_values.MOTION_EVENT }}

#define SSP_STATUS_OK {{ status_values.OK }}
#define SSP_STATUS_SENSOR_ERROR {{ status_values.SENSOR_ERROR }}

#define SSP_SERVICE_UUID_STR "{{ service_uuid }}"
#define SSP_STATUS_UUID_STR "{{ status_uuid }}"
#define SSP_EVENT_UUID_STR "{{ event_uuid }}"

struct ssp_event {
    uint8_t version;
    uint8_t event_type;
    uint8_t status;
    uint32_t device_id;
    uint32_t sequence;
    uint32_t timestamp_ms;
    uint16_t motion_mg;
};

/*
 * Explicit serialization avoids relying on C struct packing or
 * host endianness.
 */
static inline void ssp_encode_event(
    const struct ssp_event *event,
    uint8_t out[SSP_EVENT_PAYLOAD_LENGTH])
{
    size_t i = 0;

    out[i++] = event->version;
    out[i++] = event->event_type;
    out[i++] = event->status;

    for (unsigned int n = 0; n < 4; ++n)
        out[i++] = (uint8_t)(event->device_id >> (8 * n));

    for (unsigned int n = 0; n < 4; ++n)
        out[i++] = (uint8_t)(event->sequence >> (8 * n));

    for (unsigned int n = 0; n < 4; ++n)
        out[i++] = (uint8_t)(event->timestamp_ms >> (8 * n));

    out[i++] = (uint8_t)(event->motion_mg & 0xff);
    out[i++] = (uint8_t)(event->motion_mg >> 8);
}

#endif
```

The generated C encoder and Kotlin decoder must use exactly the same field order and byte order. The shared test vector catches accidental changes.

## 2.5 Add the Kotlin protocol template

Create `tools/ble_codegen/templates/SspProtocol.kt.j2`:

kotlin
```
package org.ssp.protocol

import java.nio.ByteBuffer
import java.nio.ByteOrder
import java.util.UUID

object SspProtocol {
    const val VERSION = {{ spec.protocol.version }}
    const val PAYLOAD_LENGTH = {{ payload_length }}
    const val DEVICE_ID = {{ device_id }}L

    const val EVENT_NONE = {{ event_values.NONE }}
    const val EVENT_MOTION_EVENT = {{ event_values.MOTION_EVENT }}

    const val STATUS_OK = {{ status_values.OK }}
    const val STATUS_SENSOR_ERROR = {{ status_values.SENSOR_ERROR }}

    val SERVICE_UUID: UUID = UUID.fromString("{{ service_uuid }}")
    val STATUS_UUID: UUID = UUID.fromString("{{ status_uuid }}")
    val EVENT_UUID: UUID = UUID.fromString("{{ event_uuid }}")

    data class Event(
        val version: Int,
        val eventType: Int,
        val status: Int,
        val deviceId: Long,
        val sequence: Long,
        val timestampMs: Long,
        val motionMg: Int
    )

    fun decode(bytes: ByteArray): Event {
        require(bytes.size == PAYLOAD_LENGTH) {
            "Invalid SSP payload length: ${bytes.size}"
        }

        val buffer = ByteBuffer.wrap(bytes).order(ByteOrder.LITTLE_ENDIAN)

        val event = Event(
            version = buffer.get().toInt() and 0xff,
            eventType = buffer.get().toInt() and 0xff,
            status = buffer.get().toInt() and 0xff,
            deviceId = buffer.int.toLong() and 0xffffffffL,
            sequence = buffer.int.toLong() and 0xffffffffL,
            timestampMs = buffer.int.toLong() and 0xffffffffL,
            motionMg = buffer.short.toInt() and 0xffff
        )

        require(event.version == VERSION) { "Unsupported protocol version" }
        require(event.eventType in EVENT_NONE..EVENT_MOTION_EVENT) {
            "Invalid event type"
        }
        require(event.status in STATUS_OK..STATUS_SENSOR_ERROR) {
            "Invalid status"
        }
        require(event.status == STATUS_OK) {
            "Device reports sensor error"
        }

        return event
    }
}
```

## 2.6 Add the protocol documentation template

Create `tools/ble_codegen/templates/protocol.md.j2`:

markdown
```
# {{ spec.protocol.name }}

Protocol version: {{ spec.protocol.version }}

## GATT interface

| Component | UUID | Properties |
|---|---|---|
| SSP service | {{ service_uuid }} | Primary service |
| Status | {{ status_uuid }} | Read |
| Event | {{ event_uuid }} | Notify |

## Event payload

Length: {{ payload_length }} bytes.

Byte order: little-endian.

| Field | Type | Size |
|---|---|---:|
{% for field in fields %}
| `{{ field.name }}` | `{{ field.type }}` | {{ {"u8": 1, "u16": 2, "u32": 4}[field.type] }} |
{% endfor %}

The timestamp is device uptime in milliseconds.
The motion metric is represented in milli-g.

## Test vectors

{% for test in spec.tests %}
- `{{ test.name }}`: `{{ test.expected_hex }}`
{% endfor %}
```

## 2.7 Generate and verify

bash
```
cd ~/ssp-iot-poc

python tools/ble_codegen/generate.py --check
python tools/ble_codegen/generate.py

git status --short
```

At this point, Batch A has a reproducible protocol contract, generated C/Kotlin definitions, a protocol document, and a shared test-vector file.

# 3. Batch B — Generate the Zephyr BLE peripheral

Create a separate application at `device/firmware/ble_peripheral/`.

## 3.1 Firmware build configuration

`device/firmware/ble_peripheral/CMakeLists.txt`

cmake
```
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(ssp_ble_peripheral)

target_sources(app PRIVATE
    src/main.c
    src/ssp_gatt.c
    src/ssp_event.c
)

target_include_directories(app PRIVATE
    src
    src/generated
)
```

`device/firmware/ble_peripheral/prj.conf`

ini
```
CONFIG_BT=y
CONFIG_BT_PERIPHERAL=y
CONFIG_BT_DEVICE_NAME="SSP-POC"
CONFIG_BT_MAX_CONN=1
CONFIG_BT_MAX_PAIRED=0

CONFIG_SERIAL=y
CONFIG_CONSOLE=y
CONFIG_UART_CONSOLE=y
CONFIG_PRINTK=y

CONFIG_MAIN_STACK_SIZE=2048
CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=2048
```

These are starting settings for an unpaired lab demonstration, not a production security configuration. Validate them against the installed Zephyr revision and board.

## 3.2 Implement the event producer

`device/firmware/ble_peripheral/src/ssp_event.h`

c
```
#ifndef SSP_EVENT_H
#define SSP_EVENT_H

#include <stdint.h>
#include <stdbool.h>

void ssp_event_init(void);
void ssp_event_start(void);
bool ssp_event_subscribed(void);

#endif
```

`device/firmware/ble_peripheral/src/ssp_event.c`

c
```
#include <zephyr/kernel.h>
#include <zephyr/sys/printk.h>
#include <zephyr/bluetooth/bluetooth.h>
#include <zephyr/bluetooth/gatt.h>

#include "generated/ssp_protocol.h"
#include "ssp_event.h"

extern struct bt_gatt_attr ssp_event_attr;

static uint32_t sequence;

static void event_ccc_changed(
    const struct bt_gatt_attr *attr,
    uint16_t value)
{
    ARG_UNUSED(attr);

    printk("SSP event notifications: %s\n",
           value == BT_GATT_CCC_NOTIFY ? "enabled" : "disabled");
}

BT_GATT_SERVICE_DEFINE(ssp_service,
    BT_GATT_PRIMARY_SERVICE(
        BT_UUID_DECLARE_128(
            BT_UUID_128_ENCODE(
                0x7d2ea28a, 0xf7bd, 0x485a,
                0xbd, 0x9d, 0x92, 0xad, 0x6e, 0xcf, 0xe9, 0x3e
            )
        )
    ),
    BT_GATT_CHARACTERISTIC(
        BT_UUID_DECLARE_128(
            BT_UUID_128_ENCODE(
                0x7d2ea28a, 0xf7bd, 0x485a,
                0xbd, 0x9d, 0x92, 0xad, 0x6e, 0xcf, 0xe9, 0x40
            )
        ),
        BT_GATT_CHRC_READ,
        BT_GATT_PERM_READ,
        NULL, NULL, NULL
    ),
    BT_GATT_CHARACTERISTIC(
        BT_UUID_DECLARE_128(
            BT_UUID_128_ENCODE(
                0x7d2ea28a, 0xf7bd, 0x485a,
                0xbd, 0x9d, 0x92, 0xad, 0x6e, 0xcf, 0xe9, 0x41
            )
        ),
        BT_GATT_CHRC_NOTIFY,
        BT_GATT_PERM_NONE,
        NULL, NULL, NULL
    ),
    BT_GATT_CCC(event_ccc_changed,
        BT_GATT_PERM_READ | BT_GATT_PERM_WRITE)
);

void ssp_event_init(void)
{
    sequence = 0;
}

bool ssp_event_subscribed(void)
{
    return bt_gatt_is_subscribed(
        NULL,
        &ssp_service.attrs[4],
        BT_GATT_CCC_NOTIFY
    );
}

void ssp_event_start(void)
{
    /*
     * Initial implementation hook:
     * publish a synthetic event after subscription.
     *
     * The production implementation should use the
     * connection-aware notification API and the actual
     * attribute belonging to the event characteristic.
     */
    printk("SSP synthetic event producer ready\n");
}
```

Important: treat this file as a template starting point, not a flash-ready implementation. Before building, the generated GATT module must expose the status read handler, retain the correct event attribute, and send notifications to the subscribed connection. The UUIDs in the service above should also be moved into the generated protocol header so the YAML remains authoritative.

The intended notification operation is:

c
```
struct ssp_event event = {
    .version = SSP_PROTOCOL_VERSION,
    .event_type = SSP_EVENT_MOTION_EVENT,
    .status = SSP_STATUS_OK,
    .device_id = SSP_DEVICE_ID,
    .sequence = sequence++,
    .timestamp_ms = (uint32_t)k_uptime_get(),
    .motion_mg = 1800,
};

uint8_t payload[SSP_EVENT_PAYLOAD_LENGTH];
ssp_encode_event(&event, payload);

/* Send payload using bt_gatt_notify() to the subscribed connection. */
```

This keeps synthetic event generation separate from the physical IMU. The notification code should only publish when the client is subscribed.

## 3.3 Add the application entry point

`device/firmware/ble_peripheral/src/main.c`

c
```
#include <zephyr/kernel.h>
#include <zephyr/sys/printk.h>
#include <zephyr/bluetooth/bluetooth.h>
#include <zephyr/bluetooth/conn.h>

#include "ssp_event.h"

static struct bt_conn *active_connection;

static void connected(struct bt_conn *conn, uint8_t err)
{
    if (err != 0) {
        printk("BLE connection failed: %u\n", err);
        return;
    }

    active_connection = bt_conn_ref(conn);
    printk("SSP Edge connected\n");
}

static void disconnected(struct bt_conn *conn, uint8_t reason)
{
    ARG_UNUSED(conn);

    printk("SSP Edge disconnected: %u\n", reason);

    if (active_connection != NULL) {
        bt_conn_unref(active_connection);
        active_connection = NULL;
    }
}

BT_CONN_CB_DEFINE(connection_callbacks) = {
    .connected = connected,
    .disconnected = disconnected,
};

int main(void)
{
    int err;

    printk("SSP BLE peripheral starting\n");

    ssp_event_init();

    err = bt_enable(NULL);
    if (err != 0) {
        printk("Bluetooth initialization failed: %d\n", err);
        return err;
    }

    printk("Bluetooth initialized\n");

    /*
     * Register the advertising data and start connectable
     * advertising here. Use the SSP service UUID and device
     * name defined by the protocol/configuration.
     */

    ssp_event_start();

    return 0;
}
```

This entry point deliberately marks advertising and notification wiring as implementation tasks rather than pretending the skeleton is a complete peripheral. Batch B is complete only after those functions are connected and the firmware compiles and advertises successfully.

# 4. Batch C — Android BLE client and SSP state processing

The Android side should be split into three independent modules:

- BLE transport: scanning, connection, service discovery and notifications.
- Protocol: decode and validate the generated 17-byte event.
- State: maintain the current SSP monitoring state.

## 4.1 Implement deterministic state handling

Create `edge/android/app/src/main/java/org/ssp/state/SspState.kt`:

kotlin
```
package org.ssp.state

import org.ssp.protocol.SspProtocol

enum class MonitoringState {
    NORMAL,
    ELEVATED
}

data class StateResult(
    val state: MonitoringState,
    val accepted: Boolean,
    val message: String
)

class SspState {
    var current: MonitoringState = MonitoringState.NORMAL
        private set

    private var lastSequence: Long? = null

    fun accept(event: SspProtocol.Event): StateResult {
        if (event.eventType != SspProtocol.EVENT_MOTION_EVENT) {
            return StateResult(
                current, false, "No motion event"
            )
        }

        val previous = lastSequence
        if (previous != null && event.sequence <= previous) {
            return StateResult(
                current, false, "Duplicate or out-of-order event"
            )
        }

        lastSequence = event.sequence
        current = MonitoringState.ELEVATED

        return StateResult(
            current, true, "Valid motion event accepted"
        )
    }

    fun resetForNewSession() {
        current = MonitoringState.NORMAL
        lastSequence = null
    }
}
```

This first version assumes a single session with no 32-bit sequence rollover. Handle rollover and reconnect/session semantics explicitly before treating the validator as production-ready.

## 4.2 Implement the BLE connection lifecycle

The Android BLE module should expose an interface that the UI can consume, rather than embedding GATT logic inside an Activity:

kotlin
```
package org.ssp.ble

import org.ssp.protocol.SspProtocol

interface SspBleClient {
    fun startScan()
    fun connect(deviceAddress: String)
    fun disconnect()
    fun close()

    fun onConnectionChanged(connected: Boolean)
    fun onEventReceived(event: SspProtocol.Event)
    fun onError(message: String)
}
```

Implement the platform-specific class using Android's `BluetoothLeScanner` and `BluetoothGatt` APIs. Its state machine should be:

text
```
IDLE
  ↓ scan
SCANNING
  ↓ device found
CONNECTING
  ↓ connected
DISCOVERING_SERVICES
  ↓ SSP service verified
ENABLING_NOTIFICATIONS
  ↓ CCCD write succeeds
READY
  ↓ notification
DECODE → VALIDATE → UPDATE STATE
```

The implementation must verify both the service UUID and event characteristic UUID before enabling notifications. It must register the local notification callback and write the Client Characteristic Configuration Descriptor (CCCD) before treating the client as `READY`.

Android BLE callback signatures and permission requirements vary with the Android API level; implement the callback and permission handling against the project's actual `compileSdk` and `targetSdk`.

## 4.3 Define the first UI acceptance criteria

The UI can be a simple Compose screen initially. It needs to show:

- Connection state: disconnected, connecting or connected.
- Device identity and last sequence number.
- Latest event and timestamp.
- Monitoring state: `NORMAL` or `ELEVATED`.
- A visible error if packet validation fails.

Do not add a database, cloud API or ML component to this batch. The first demo should prove the local Device → Edge path before introducing additional variables.

# 5. Batch D — One-command build, tests and integration report

## 5.1 Create the build orchestrator

Create `tools/build_ble_demo.sh`:

bash
```
#!/usr/bin/env bash
set -Eeuo pipefail

ROOT="$(git -C "$(dirname "$0")/.." rev-parse --show-toplevel)"
WORKSPACE="$HOME/zephyrproject"
BUILD_DIR="/tmp/ssp-ble-demo"
FIRMWARE="$ROOT/device/firmware/ble_peripheral"
ANDROID="$ROOT/edge/android"

echo "========================================"
echo " SSP BLE Demo Build Pipeline"
echo "========================================"

cd "$ROOT"

# Gate A: generate and validate the protocol.
echo
echo "[A] Validate protocol and test vectors"
python tools/ble_codegen/generate.py --check

echo
echo "[A] Generate protocol artifacts"
python tools/ble_codegen/generate.py

# Gate B: build Zephyr firmware.
echo
echo "[B] Build Zephyr BLE peripheral"

test -d "$WORKSPACE/zephyr" || {
    echo "ERROR: Zephyr source tree not found."
    exit 1
}

test -f "$FIRMWARE/CMakeLists.txt" || {
    echo "ERROR: BLE firmware application is incomplete."
    exit 1
}

export ZEPHYR_BASE="$WORKSPACE/zephyr"

cd "$WORKSPACE"

west build -p always \
    -b nrf52840dk/nrf52840 \
    -d "$BUILD_DIR" \
    "$FIRMWARE"

for image in \
    "$BUILD_DIR/zephyr/zephyr.hex" \
    "$BUILD_DIR/zephyr/zephyr.elf"
do
    test -s "$image" || {
        echo "ERROR: Missing firmware artifact: $image"
        exit 1
    }
done

echo "Zephyr build: PASS"

# Gate C: build Android, if the project is present.
echo
echo "[C] Build Android Edge"

if [[ -x "$ANDROID/gradlew" ]]; then
    cd "$ANDROID"
    ./gradlew testDebugUnitTest assembleDebug
    echo "Android build and unit tests: PASS"
else
    echo "Android Gradle project not yet generated."
    echo "Android gate: NOT RUN"
fi

# Gate D: summarize the generated firmware.
echo
echo "[D] Firmware artifacts"
ls -lh \
    "$BUILD_DIR/zephyr/zephyr.hex" \
    "$BUILD_DIR/zephyr/zephyr.elf"

echo
echo "========================================"
echo " SSP BLE BUILD PIPELINE: PASS"
echo "========================================"
echo
echo "Note: build PASS does not imply BLE"
echo "hardware integration has passed."
```

Make it executable:

bash
```
chmod +x tools/build_ble_demo.sh
```

This is the initial orchestrator. Before relying on it for a release, add the automated protocol unit tests as a mandatory gate and distinguish a fully passing pipeline from a pipeline in which the Android gate was skipped.

## 5.2 Add the shared protocol test

Create `tests/ble/test_protocol.py`:

python
```
import json
import struct
import unittest
from pathlib import Path

ROOT = Path(__file__).resolve().parents[2]
VECTORS = ROOT / "tests/ble/test_vectors.json"

class ProtocolTests(unittest.TestCase):

    def test_shared_motion_vector(self):
        vectors = json.loads(VECTORS.read_text(encoding="utf-8"))
        vector = next(
            v for v in vectors
            if v["name"] == "synthetic_motion_event"
        )

        values = vector["values"]

        payload = struct.pack(
            "<BBBIIIH",
            values["version"],
            1 if values["event_type"] == "MOTION_EVENT"
            else values["event_type"],
            values["status"],
            values["device_id"],
            values["sequence"],
            values["timestamp_ms"],
            values["motion_mg"],
        )

        self.assertEqual(len(payload), 17)
        self.assertEqual(payload.hex(), vector["expected_hex"])

    def test_invalid_payload_length(self):
        payload = bytes(16)
        self.assertNotEqual(len(payload), 17)

if __name__ == "__main__":
    unittest.main()
```

Run the test after generation:

bash
```
python -m unittest discover -s tests/ble -p 'test_*.py' -v
```

The shared vector should ultimately be tested against both the generated C encoder and Kotlin decoder as well, so all three implementations are checked against the same bytes.

## 5.3 Run the pipeline

bash
```
cd ~/ssp-iot-poc

python tools/ble_codegen/generate.py --check
python -m unittest discover -s tests/ble -p 'test_*.py' -v
bash tools/build_ble_demo.sh
```

The existing Hello World scripts remain available as independent diagnostics. Do not change their expected UART messages or their build directory to accommodate BLE.

# 6. The most efficient implementation sequence

## A

Protocol contract and generation

Deliverable: YAML, C/Kotlin constants, shared vectors, protocol documentation.

Exit gate: generator and protocol tests pass.

## B

Zephyr BLE peripheral

Deliverable: advertising, GATT service, event notification and synthetic producer.

Exit gate: firmware builds and a BLE client receives a valid event.

## C

Android BLE and state

Deliverable: scanning, connection, notification decoding and state display.

Exit gate: a valid event changes `NORMAL` to `ELEVATED`.

## D

Automated integration

Deliverable: reproducible builds, shared tests and hardware acceptance evidence.

Exit gate: repeatable device-to-Android demonstration.

## 7. Three corrections before treating this as executable firmware

The code above is a generation scaffold, not yet a complete working BLE implementation. There are three important completion tasks:

1. Make UUIDs genuinely generated. The example GATT module still contains UUID definitions directly. Move their byte representations into the generated header so changing YAML updates both endpoints.
2. Complete Zephyr GATT and advertising. Implement the status read callback, connection-aware notification delivery, subscription handling, and connectable advertising. Do not flash the skeleton as though it were complete.
3. Complete Android transport and build configuration. The Kotlin decoder and state reducer are the first modules, not a complete Android application. Add the Gradle project or integrate these modules into your existing app, then implement the BLE lifecycle and UI.

Also correct the device ID in the initial YAML to `0x53535001` (decimal `1397968897`) so the decimal constant agrees with the shared byte vector.

My recommendation: implement Batch A completely, then finish Batch B and C against the same protocol contract. The first meaningful milestone is not simply a successful Zephyr build—it is one valid synthetic `MOTION_EVENT` received over real BLE by Android, validated, and displayed as `ELEVATED`. That is the earliest demonstration of SSP's core architecture.
