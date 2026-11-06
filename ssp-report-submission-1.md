# SSP-IoT-POC — Device–BLE–Edge Communication Implementation Report

**Deliverable:** IoT Lab — Deliverable 1\
**Document Status:** Final Version\
**Project:** SSP-IoT-POC\
**Target Hardware:** Nordic nRF52840\
**Validation Client:** nRF Connect on iPhone\
**Target Edge Application:** Android SSP Application\
**Communication Protocol:** SSP Protocol v1

---

## Executive Summary

SmartSecurePerimeter (SSP) is an IoT monitoring platform based on a Nordic nRF52840 device and an Android Edge application.

This deliverable focuses on implementing and validating BLE communication between the electronic device and the Edge layer.

The implemented solution provides BLE advertising, connection management, GATT-based communication, notification-based event transmission and Protocol v1 serialization.

Validation was performed using nRF Connect on iPhone as the BLE validation client, supported by firmware UART diagnostics.

The implementation demonstrates successful device discovery, connection establishment, disconnection detection, automatic advertising recovery, reconnection and transmission of 17-byte SSP event payloads.

The Protocol v1 payload structure has been verified directly against the firmware serializer and validated against captured runtime data.

---

## 1\. Objective

Implement and validate BLE communication between the SSP electronic device (nRF52840) and an Edge application.

The implementation focuses on:

- Device discovery through BLE advertising.
- Establishment and management of BLE connections.
- GATT service and characteristic communication.
- Notification-based transmission of SSP events.
- Protocol v1 payload serialization and data transfer.
- Connection recovery and reconnection.

The objective is to demonstrate reliable device-to-Edge communication using BLE as the transport layer.

## 2\. System Architecture

The implementation follows this communication architecture:

text

```text
         SSP DEVICE (nRF52840)
                  |
          Sensor Manager
                  |
          Motion Detector
                  |
          SSP Event Subsystem
                  |
          Protocol v1 Encoder
                  |
             BLE GATT
                  |
          Notification Characteristic
                  |
                  v
          EDGE APPLICATION
           (Android SSP App)
                  |
          BLE Event Reception
                  |
          Protocol Decoding
                  |
          Event Presentation
```

**Validation configuration:** nRF Connect on iPhone is currently used as the BLE validation client. The Android SSP application is the intended Edge implementation.

## 3\. BLE Design

The implementation uses the following BLE mechanisms:

| BLE Component | Purpose |
| --- | --- |
| Advertising | Makes the SSP device discoverable to BLE clients. |
| Connection | Establishes a communication session between the nRF52840 and the Edge Client. |
| GATT | Provides the service and characteristic structure for SSP communication. |
| Custom GATT Service | Exposes SSP functionality to the Edge Client. |
| Notification Characteristic | Transmits SSP event payloads from the device to the Edge Client. |
| CCCD | Allows the Edge Client to enable or disable notifications. |
| Connection Callbacks | Detect connection and disconnection events. |
| Advertising Restart | Restores device discoverability following disconnection. |

### Communication Flow

1. The nRF52840 starts BLE advertising using the device name `SSP-POC`.
2. The Edge Client scans and discovers the device.
3. The Edge Client establishes a BLE connection.
4. GATT service and characteristic discovery identifies the SSP communication interface.
5. The Edge Client enables notifications through the CCCD.
6. The device generates SSP events and serializes them using Protocol v1.
7. The firmware transmits event payloads through BLE notifications.
8. The Edge Client receives and decodes the payload.

Communication currently uses device-to-Edge BLE notifications.

## 4\. Implementation Summary

The following functionality has been implemented and observed:

| Component | Implementation | Status |
| --- | --- | --- |
| Device initialization | nRF52840 firmware startup | Validated |
| BLE advertising | SSP-POC advertising | Validated |
| Device discovery | iPhone BLE scanner | Validated |
| BLE connection | GATT connection | Validated |
| GATT service discovery | Existing BLE client implementation | Implemented |
| Disconnection detection | Connection callback | Validated |
| Advertising recovery | Automatic restart | Validated |
| Reconnection | New connection established | Validated |
| Notification subscription | Client-side enablement | Observed |
| SSP event generation | Simulated motion event | Observed |
| Protocol v1 serialization | Fixed 17-byte payload | Validated |
| BLE notification transmission | Five-second interval | Validated |

## 5\. BLE Regression Test Results

Testing was performed using nRF Connect on iPhone.

### 5.1 Connection and Disconnection

The device successfully established a BLE connection and detected the deliberate iPhone disconnection.

Observed UART:

text

```text
[CONNECT] DISCONNECTED count=1 reason=19
```

Bluetooth disconnect reason `0x13` indicates Remote User Terminated Connection, consistent with the manual disconnection initiated through the iPhone.

**Result: PASS**

### 5.2 Automatic Advertising Recovery

Following disconnection, the firmware automatically restarted advertising.

Observed UART:

text

```text
[ADV] ADV_RESTART_SCHEDULED
[ADV] ADV_RESTART_BEGIN
[ADV] ADV_START attempt=2
SSP advertising started: SSP-POC
[ADV] ADV_START result=0
[ADV] ADV_RESTART_END
```

Advertising restarted successfully.

**Result: PASS**

### 5.3 BLE Reconnection

The iPhone successfully reconnected to the device.

text

```text
[CONNECT] CONNECTED count=2
```

The complete connection recovery sequence was demonstrated:

- Initial connection.
- Client-initiated disconnection.
- Automatic advertising restart.
- Successful reconnection.

**Result: PASS**

### 5.4 Notification Delivery

After reconnection, notifications were enabled through the iPhone scanner.

Observed UART:

text

```text
[MONITOR] Notification session started

MOTION detected=1 motion=2100

[NOTIFY] #14 len=17
[NOTIFY] #15 interval=5.000s len=17
[NOTIFY] #16 interval=5.000s len=17
[NOTIFY] #17 interval=5.000s len=17
```

Notification session summary:

text

```text
Duration : 21s
Count    : 4
Avg Int. : 5.000s
Errors   : 0
```

Results:

- Four notification transmissions reported successful.
- Consistent five-second transmission interval.
- No reported notification errors.
- Notification delivery resumed after reconnection.

**Result: PASS — Firmware-side notification transmission.**

### 5.5 Notification Disable and Firmware Continuity

After disabling notifications through the iPhone scanner, the firmware continued reporting heartbeat messages.

Observed UART:

text

```text
[ALIVE] ALIVE hb=1367 adv=2 conn=2 disconn=1
[ALIVE] ALIVE hb=1368 adv=2 conn=2 disconn=1
```

This demonstrates continued firmware operation and an apparently maintained BLE connection after the notification monitoring session ended.

Explicit confirmation of the peripheral's GATT CCCD subscription state remains outstanding.

**Result: PARTIALLY VALIDATED**

## 6\. Protocol v1 Validation

The SSP firmware implements a fixed-length, 17-byte Protocol v1 event payload.

The payload structure has been verified directly against the firmware serializer implementation in `ssp_protocol.h` and validated against captured runtime payloads emitted by the device.

### 6.1 Payload Data Structure

The firmware event structure is:

c

```c
struct ssp_event {
    uint8_t version;
    uint8_t event_type;
    uint8_t status;
    uint32_t device_id;
    uint32_t sequence;
    uint32_t timestamp_ms;
    uint16_t motion_mg;
};
```

The firmware defines:

c

```c
#define SSP_EVENT_PAYLOAD_LENGTH 17
```

The serializer explicitly implements a fixed-length, little-endian wire representation.

### 6.2 Validated Payload Layout

| Byte Offset | Size | Field | Encoding |
| --- | --- | --- | --- |
| 0 | 1 byte | Protocol Version | uint8 |
| 1 | 1 byte | Event Type | uint8 |
| 2 | 1 byte | Status | uint8 |
| 3–6 | 4 bytes | Device ID | uint32, little-endian |
| 7–10 | 4 bytes | Sequence Number | uint32, little-endian |
| 11–14 | 4 bytes | Timestamp (ms) | uint32, little-endian |
| 15–16 | 2 bytes | Motion Value (mg) | uint16, little-endian |
| **Total** | **17 bytes** | **SSP Protocol v1 Event** |  |

### 6.3 Runtime Payload Validation

Captured device payload:

text

```text
01 01 00 01 50 53 53 0E 00 00 00 68 0A 68 00 34 08
```

Decoded fields:

| Byte Offset | Captured Value | Interpretation |
| --- | --- | --- |
| 0 | `01` | Protocol Version = 1 |
| 1 | `01` | MOTION\_EVENT |
| 2 | `00` | SSP\_STATUS\_OK |
| 3–6 | `01 50 53 53` | Device ID |
| 7–10 | `0E 00 00 00` | Sequence Number = 14 |
| 11–14 | `68 0A 68 00` | Timestamp (ms) |
| 15–16 | `34 08` | Motion = 2100 mg |

Motion value decoding:

text

```text
0x0834 = 2100 mg
```

### 6.4 Transmission Validation

Subsequent runtime notifications demonstrated sequence progression:

text

```text
0E → 0F → 10 → 11
```

Observed transmission characteristics:

| Parameter | Result |
| --- | --- |
| Protocol | SSP Protocol v1 |
| Payload length | 17 bytes |
| Byte order | Little-endian |
| Notification interval | 5 seconds |
| Events transmitted | 4 |
| Transmission errors | 0 |

The payload structure, field serialization and motion value were verified against the firmware implementation and captured runtime output.

**Result: PASS — Protocol v1 serialization and firmware-side notification transmission validated.**

## 7\. Event Generation Pipeline Validation

The simulated event generation architecture produced the expected motion output.

Observed UART:

text

```text
MOTION detected=1 motion=2100
```

Expected simulated calculation:

text

```text
1200 + 900 = 2100
```

The observed motion value matches the expected simulation output.

Current processing flow:

text

```text
Simulated IMU
      |
Sensor Manager
      |
Motion Detector
      |
SSP Event Subsystem
      |
Protocol v1 Encoder
      |
BLE Notification
```

This confirms the expected simulated motion value and its presence in the firmware notification path.

**Result: PASS — Simulated event generation pipeline.**

### Android Edge Application Status

The Android SSP application already implements:

- BLE discovery.
- BLE connection.
- GATT service discovery.
- Notification subscription.
- Protocol v1 decoding implementation.

Android-side event presentation and complete end-to-end payload reception evidence remain outstanding.

The iPhone validation client is used to test BLE communication independently of the final Android Edge interface.

## 8\. Limitations

The following limitations remain:

1. **CCCD verification:** Explicit confirmation that the nRF52840 receives and processes notification enable/disable configuration changes has not yet been captured.
2. **End-to-end Edge validation:** Firmware-side notification transmission is confirmed. Android-side payload reception, decoding and event presentation still require formal validation.
3. **Sensor simulation:** Motion events currently use simulated IMU inputs. Physical IMU acquisition and hardware-driven motion detection have not been demonstrated.
4. **GPS integration:** GPS acquisition, NMEA parsing and SSP location object validation have not yet been demonstrated.

## 9\. Next Steps

Complete explicit GATT CCCD notification enable/disable verification, finalize the Deliverable 1 evidence package, and validate Android-side event reception and Protocol v1 decoding. Subsequently, proceed with GPS and IMU hardware integration to enable real sensor-driven SSP events.

## 10\. Conclusion

The SSP-IoT-POC implementation demonstrates successful device-to-Edge BLE communication, including device discovery, connection establishment, disconnection detection, automatic advertising recovery, reconnection and Protocol v1 notification transmission.

The simulated event generation pipeline produces the expected motion value of 2100, with successful 17-byte notification transmission at five-second intervals.

The Protocol v1 payload structure and little-endian serialization have been verified directly against the firmware implementation and captured runtime output.

The BLE communication implementation has passed the observed regression tests. Explicit CCCD verification, complete Android Edge validation and physical sensor integration remain outstanding.

**Overall status:** Device–BLE communication implemented and regression-tested. Protocol v1 serialization validated. End-to-end Android validation and physical sensor-driven operation remain pending.

---

## Appendix A — Supporting Evidence

The following evidence should accompany the report in the GitHub repository.

| Appendix | Evidence | Purpose |
| --- | --- | --- |
| A1 | BLE device discovery screenshot | Demonstrates SSP-POC advertising and discovery. |
| A2 | Connected device screenshot | Demonstrates successful BLE connection. |
| A3 | Notification-enabled screenshot | Demonstrates client-side notification subscription. |
| A4 | UART connection and advertising logs | Demonstrates connection lifecycle and advertising recovery. |
| A5 | UART notification logs | Demonstrates event transmission and five-second interval. |
| A6 | Protocol v1 payload example | Demonstrates the captured 17-byte event payload. |

**Supporting evidence:** Screenshots and UART log excerpts are maintained separately from the main report.

## Suggested GitHub structure

text

```text
SSP-IoT-POC/
│
├── docs/
│   └── deliverable-1/
│       ├── SSP_Device_BLE_Edge_Communication_Report.md
│       │
│       └── evidence/
│           ├── A1_ble_discovery.png
│           ├── A2_ble_connected.png
│           ├── A3_notifications_enabled.png
│           ├── A4_connection_recovery.log
│           ├── A5_notification_delivery.log
│           └── A6_protocol_v1_payload.log
```

## Appendix B — Development Environments and Toolset

The SSP-IoT-POC development environment combines embedded firmware development, BLE communication testing and Android application development. The device firmware targets the Nordic nRF52840 and is developed using the Zephyr RTOS ecosystem, including the `west` build and flash tools. UART serial logging is used for firmware diagnostics and runtime validation. BLE discovery, connection management, GATT service inspection and notification testing are performed using nRF Connect on iPhone. The Android SSP Edge application is developed using Android Studio and provides the client-side BLE communication and Protocol v1 decoding implementation. Git and GitHub are used for source-code version control, project documentation and deliverable management.

| Area | Environment / Tool |
| --- | --- |
| Embedded hardware | Nordic nRF52840 |
| Firmware platform | Zephyr RTOS |
| Build and flash | West (`west`) |
| Firmware diagnostics | UART Serial Monitor |
| BLE validation | nRF Connect on iPhone |
| Android development | Android Studio |
| Edge application | Android SSP Application |
| Version control | Git |
| Repository and documentation | GitHub |
