The results are useful: the protocol contract is a promising starting point, and the generated-code placeholders are confirmed empty. We can now move from inventory to implementation, but I recommend resolving a few protocol details first so that the generator, Zephyr firmware, and Android client all implement exactly the same contract.

## 1\. Protocol contract assessment

| Area | Assessment | Required action |
| --- | --- | --- |
| Protocol version | `1` | Keep; validate unsupported versions |
| Payload length | 17 bytes | Correct for the declared field layout |
| Byte order | Little-endian | Keep and test explicitly |
| Device ID | `1397960705` = `0x53535001` | Correct; use consistently |
| GATT UUIDs | Three UUIDs defined | Validate uniqueness and generate declarations |
| Event and status enums | Defined | Reject unknown values |
| Timestamp | Milliseconds since boot | Keep; document 32-bit wraparound |
| Motion metric | Provisional milli-g | Keep provisional; define the synthetic value precisely |
| Synthetic event schedule | 3 s initial delay, 5 s repeat interval | Implement and test |
| Subscription requirement | Explicitly enabled | Require a notification subscriber before delivery |

The test vector is internally consistent. Its 17 bytes encode the declared values in little-endian order, including device ID `0x53535001` and motion metric `1800`.

## 2\. Three important decisions before generation

### A. Define the status characteristic's payload

The YAML defines a readable status characteristic, but it does not define the bytes returned by that characteristic.

For the first demo, I recommend a compact, fixed one-byte status response:

- `0x00` — device ready
- `0x01` — sensor error

This can reuse the existing status enum. Document that the characteristic reports the current device status, whereas the `status` field in an event reports the status associated with that event.

### B. Make notification behavior unambiguous

The event characteristic has `notify` but no `read`, which is appropriate for the initial event-driven demo. The firmware should:

- Start connectable advertising.
- Track the active BLE connection.
- Track notification subscription using the event characteristic's actual GATT attribute.
- Send the synthetic event only when notifications are enabled.
- Increment the sequence counter for each generated event, using unsigned 32-bit wraparound.

For the first version, we should explicitly decide whether events generated without a subscriber are discarded or queued. I recommend discarding them, because this is a periodic synthetic demonstration rather than a reliable event store. Document this behavior so Android does not expect replay after reconnecting.

### C. Define sequence validation at Android

A simple comparison such as `incomingSequence <= previousSequence` fails when the unsigned 32-bit counter wraps. Android should also reset or re-establish its sequence baseline when a new device session begins.

For the initial demo, use modular sequence arithmetic, and treat reconnect/session handling separately from packet validity. The sequence counter alone cannot reliably identify every device restart.

## 3\. Recommended small amendment to the YAML

Add explicit status-read and event-delivery semantics to `protocol/ssp_ble_v1.yaml` before generating the protocol artifacts.

The following is a proposed amendment, not a replacement for your existing YAML:

yaml
```
gatt:
  characteristics:
    status:
      uuid: "7d2ea28a-f7bd-485a-bd9d-92ad6ecfe940"
      properties: [read]
      permissions: [read]
      value:
        encoding: u8
        length: 1
        values:
          OK: 0
          SENSOR_ERROR: 1

    event:
      uuid: "7d2ea28a-f7bd-485a-bd9d-92ad6ecfe941"
      properties: [notify]
      permissions: [none]

semantics:
  timestamp_ms: "Unsigned 32-bit milliseconds since boot; wraps modulo 2^32"
  motion_mg: "Synthetic milli-g value for transport testing; physical interpretation remains provisional"
  sequence: "Unsigned 32-bit counter incremented for each generated event; wraps modulo 2^32"
  event_delivery: "Events are not queued or replayed. Events generated without an active notification subscription are discarded. Sequence increments for every generated event."
  status_characteristic: "One-byte read of current device status: OK=0, SENSOR_ERROR=1"
```

Merge these additions into the existing document, preserving the current UUIDs and payload definition.

The proposed behavior matches Zephyr's GATT model: notifications require a characteristic configured for notification and a client subscription, which the server can check using `bt_gatt_is_subscribed()`. docs.zephyrproject.org

+1

## 4\. Batch A: implement the generator as one coherent unit

Your next implementation should fill the existing generator, templates, and tests together rather than create more scripts.

| Deliverable | Required behavior |
| --- | --- |
| `tools/ble_codegen/generate.py` | Load and validate YAML; generate all outputs |
| `tools/ble_codegen/requirements.txt` | Pin or constrain the generator dependencies |
| C template | Generate UUIDs, enums, payload constants and encoding helpers |
| Kotlin template | Generate matching constants, models and strict decoder |
| Documentation template | Document the wire format and GATT contract |
| `tests/ble/test_vectors.json` | Contain the canonical 17-byte vector |
| `tests/ble/test_protocol.py` | Test encoding, decoding and rejection cases |

The generator should reject:

- Incorrect payload length or unsupported field types.
- Duplicate UUIDs or invalid UUID strings.
- Duplicate enum values or values outside the field's numeric range.
- Missing fields, overlapping offsets or inconsistent payload layouts.
- Test vectors whose encoded bytes do not match `expected_hex`.

It should also implement a genuine `--check` mode that compares generated output with the files on disk and exits nonzero when they are stale.

One important design rule: the generated C and Kotlin code must derive their UUIDs and protocol constants from the YAML, not duplicate hard-coded values in the templates.

## 5\. Batch A verification commands

After implementing the batch, the intended verification sequence is:

bash
```
cd ~/ssp-iot-poc

python -m venv .venv-ble-codegen
source .venv-ble-codegen/bin/activate

python -m pip install -r tools/ble_codegen/requirements.txt

python tools/ble_codegen/generate.py
python tools/ble_codegen/generate.py --check

python -m pytest -q tests/ble/test_protocol.py

python tools/ble_codegen/generate.py
python tools/ble_codegen/generate.py --check

git diff --check
git status --short
```

The final generation and freshness check should pass without modifying generated files. For stronger determinism evidence, compare the generated files before and after a clean regeneration.

Do not proceed to the firmware implementation until the contract, generated artifacts and protocol tests agree.

## 6\. The overall priority remains BLE-first

1. Batch A — Freeze and verify the protocol. YAML, generator, C/Kotlin definitions and shared test vectors.
2. Batch B — Prove the Zephyr BLE peripheral. Compile, advertise, connect, discover GATT, subscribe and receive synthetic notifications.
3. Batch C — Prove the Android Edge. Connect, decode, validate and display `NORMAL → ELEVATED`.
4. Batch D — Automate and demonstrate. One-command builds, unit tests, physical BLE integration and reproducible evidence.

Keep the existing Hello World application and its two passing scripts unchanged.

My recommendation: amend the YAML with the missing status and delivery semantics, then implement Batch A in one pass. Once Batch A passes, move directly to the Zephyr peripheral using synthetic events. That gets you to the first visible SSP demonstration without waiting for physical IMU integration.
