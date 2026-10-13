 ### Why Nordic fits this project better

 Given the project specification, the **Nordic nRF52840-DK will be the primary hardware** rather than a generic Arduino board.

 The key reason is that the assignment is explicitly designed around **BLE + Nordic nRF52840** and that board minimizes integration risk and gives you a much cleaner path to the final POC.

 | Requirement | Nordic nRF52840-DK | Typical Arduino |
| --- | --- | --- |
| BLE | **Native, excellent fit** | Depends heavily on board |
| Course-provided | **Yes** | Not necessarily |
| BLE → Android | **Directly supported** | Possible, but more setup |
| Zephyr | **Strong support** | Possible depending on board |
| PlatformIO | Supported | Supported |
| Power management | Excellent | Board-dependent |
| Sensors | Easy to add | Easy to add |
| Debugging | **Excellent onboard tools** | Board-dependent |
| Matches assignment | **Exactly** | Less directly |
| Risk of library problems | Lower if using Zephyr | Higher according to your course notes |
| Future POC | **Very suitable** | Suitable if BLE-capable |

Your assignment actually gives you a particularly strong clue:

 > "During the course, we lend each group a Nordic nRF52840-DK"

 and later recommends:

 > **Zephyr**

 while explicitly saying:

 > **Arduino (deprecated -- Errors in BLE library since 2024)**

 So I would **not make Arduino the foundation of the project**.

 ## Recommended hardware architecture

 I'd build your POC around this:

```
                 DEVICE
        ┌──────────────────────┐
        │  Nordic nRF52840-DK  │
        │                      │
        │  Sensor(s)           │
        │   ↓                  │
        │  Firmware / Zephyr   │
        │   ↓                  │
        │  BLE GATT            │
        └──────────┬───────────┘
                   │
                 BLE
                   │
                   ▼
        ┌──────────────────────┐
        │    Android Phone     │
        │                      │
        │  Kotlin BLE client   │
        │        ↓             │
        │  Data processing     │
        │        ↓             │
        │  HTTPS / MQTT        │
        └──────────┬───────────┘
                   │
                Internet
                   │
                   ▼
        ┌──────────────────────┐
        │        Cloud         │
        │                      │
        │ Backend/API          │
        │ Database             │
        │ ML                   │
        │ Dashboard            │
        └──────────────────────┘
```

 That architecture maps almost one-to-one onto the marking requirements.

 ### For the actual POC, I'd use

 **Device**

 - Nordic nRF52840-DK
- 1–2 simple sensors initially
- Zephyr
- BLE GATT service
- Custom characteristic for your sensor data

 **Edge**

 - Android phone
- Kotlin
- Native Android BLE APIs

 **Cloud**

 - MQTT or HTTP
- A simple backend
- Time-series database or PostgreSQL
- Web dashboard
- ML model if your project needs prediction/detection

---

 ## One important distinction: Nordic board vs "Arduino"

 If by **Arduino** you mean an Arduino Uno/Nano-class board, I would definitely not choose it for this assignment.

 If you mean an **Arduino-compatible nRF52840 board**, that's different. The hardware itself can be perfectly capable of doing the project, because the nRF52840 provides BLE.

 But you're still taking on an unnecessary software/toolchain question when the course already provides:

 **nRF52840-DK → Zephyr → Android BLE**

 That's the path I'd follow.

 In fact, you can still use **Arduino-style programming concepts** if your team prefers them, but I'd make the actual firmware framework **Zephyr**.

 ## The bigger opportunity: design the POC around the evaluation

 Your project has three major technical interfaces:

```
Sensor → BLE → Android → Internet → Cloud → Dashboard/ML
```

 I'd make each interface independently demonstrable.

 For example, define a simple BLE packet such as:

```
Byte 0      Message type
Byte 1      Sensor ID
Bytes 2-3   Sensor value
Bytes 4-7   Timestamp
Byte 8      Sequence number
```

 Then your Android application converts that into something like:

```
{
  "device_id": "device_01",
  "sensor": "temperature",
  "value": 23.7,
  "timestamp": 1727000000,
  "sequence": 124
}
```

 and sends it to your backend.

 This is particularly useful because your report explicitly requires a **BLE Data Frame** and **Edge-to-Cloud Data Format**. You can therefore demonstrate that your protocol was intentionally designed rather than simply transmitting arbitrary values.

 ## What I would buy

 You probably don't need another development board initially.

 I'd spend the hardware budget on **sensors and accessories for the nRF52840-DK**:

 - Temperature/humidity sensor
- Accelerometer/IMU
- Light sensor
- Push button
- LEDs
- Breadboard
- Jumper wires
- Appropriate breakout boards

 The exact sensor depends on your project's use case.

 For example, if your project were **predictive maintenance**, I'd choose an accelerometer/IMU. If it's **environmental monitoring**, temperature/humidity/light makes more sense.

 ### My recommendation

 **Use the course-provided Nordic nRF52840-DK + Zephyr as your Device layer.**

 Don't buy an Arduino as your main platform unless you have a specific reason to do so.

 The Nordic route gives you:

 **nRF52840-DK → Zephyr → BLE GATT → Android/Kotlin → MQTT/HTTP → Database → Dashboard/ML**

 which is almost exactly what your assignment asks you to demonstrate.

 If you tell me **what IoT product/use case your group is considering**, I can also propose a concrete **nRF52840 + sensor + Android \+ cloud architecture**, including the exact sensors I'd choose and a BLE packet format suitable for your report.
