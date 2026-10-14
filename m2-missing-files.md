Good. The inventory identifies 11 zero-byte files. We should not populate all of them indiscriminately; some are placeholders, while others are required to build and test the Android application.

## 1\. Recommended implementation order

| Priority | File | Recommendation |
| --- | --- | --- |
| 1 | `edge/android/settings.gradle.kts` | Implement Gradle project configuration |
| 2 | `edge/android/build.gradle.kts` | Configure Android and Kotlin plugins |
| 3 | `edge/android/app/build.gradle.kts` | Configure SDK, dependencies, and tests |
| 4 | `edge/android/app/src/main/AndroidManifest.xml` | Declare app and BLE permissions |
| 5 | `edge/android/app/src/test/java/org/ssp/protocol/SspProtocolTest.kt` | Add protocol decoder tests |
| 6 | `edge/android/app/src/test/java/org/ssp/state/SspStateTest.kt` | Add state-machine tests |
| 7 | `edge/android/app/src/main/java/org/ssp/ble/AndroidSspBleClient.kt` | Implement the Android BLE state machine |
| 8 | `device/firmware/ble_peripheral/src/ssp_gatt.c` | Implement Zephyr GATT handlers and notifications |
| 9 | `device/firmware/ble_peripheral/Kconfig` | Populate only if project-specific options are needed |
| 10 | `edge/android/gradlew` | Generate a real Gradle wrapper |
| 11 | `edge/android/app/src/main/java/org/ssp/ui/.gitkeep` | Keep empty until the UI is implemented, or remove later |

## 2\. Important dependencies before implementing the BLE client

Your Android BLE state machine must handle several details beyond the basic connection sequence:

- Scan for the SSP service UUID and report discovered devices to the UI.
- Connect using `BluetoothGatt`.
- Discover services and verify the expected SSP service and event characteristic.
- Enable local notifications with `setCharacteristicNotification()`.
- Write the Client Characteristic Configuration Descriptor (CCCD) and wait for successful completion before entering `READY`.
- Decode notifications using `SspProtocol.decode()`, validate the event, and update `SspState`.
- Handle disconnection, timeouts, permission errors, and cleanup without leaking GATT connections.

One important distinction: enabling notifications locally is not sufficient. The client must also complete the CCCD descriptor write successfully.

## 3\. What I recommend doing next

Start with the three Gradle files, followed by the manifest. That establishes the Android build environment before we write platform-specific code.

We should also verify that the existing `SspProtocol.kt` and the proposed `SspState.kt` and `SspBleClient.kt` agree on their APIs before implementing the Android client.

Next file: `edge/android/settings.gradle.kts`.

Paste your proposed content, or I can generate a compatible initial version for this repository. We will continue validating each file individually, as agreed, without adding unnecessary diagnostic steps.
