 # IOT Lab Guidelines Summary
 ## 1\. Overall project

 - Work in **groups of 3**.
- Your group acts as an **IoT company** hired to develop a complete IoT system.
- The system follows this architecture:

 **Device (Sensors + PCB) → Edge (Android/Raspberry Pi/Laptop) → Cloud (Backend + Database + Frontend \+ optional ML)**

 - A small-scale implementation or **simulation** is acceptable if the complete theoretical system cannot be physically implemented.
- You are expected to apply knowledge from programming, project management, AI/ML, etc.

 ## 2\. The three system layers

 ### Device — Sensor + PCB

 The device:

 - Uses sensors such as temperature, humidity, light, sound, accelerometer, or buttons.
- Uses a microcontroller to read/process sensor data.
- Communicates with the Edge using **Bluetooth Low Energy (BLE)**.
- The course provides each group with a **Nordic nRF52840-DK**.

 For development, you can use:

 - PlatformIO
- Nordic tools/SDK
- Zephyr
- Mbed
- Arduino

 **Important:** The guidelines state that Arduino is deprecated for this project because of BLE-library errors since 2024. **Zephyr is the recommended direction.**

 ### Edge

 The Edge receives data from the PCB over BLE and can process it before sending it to the Cloud.

 Possible Edge platforms:

 - **Android phone** — recommended
- Raspberry Pi
- Laptop
- For Apple users, an iPhone + Mac can also be used.

 For Android:

 - **Kotlin is recommended**, though Java is allowed.
- Android Studio, Flutter, Xamarin, or Ionic can be used.

 BLE functionality must cover:

 1. Scanning for the PCB.
2. Connecting to it.
3. Discovering GATT services/characteristics.
4. Enabling notifications.
5. Receiving sensor data continuously.

 For modern Android (12/API 31+), the report provides specific permissions for `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`, and location access.

 ### Cloud

 The Cloud layer should:

 - Receive data from the Edge.
- Store the data in a database.
- Provide a way to visualize the data.
- Optionally perform computational tasks such as **prediction or detection using ML**.

 You may use:

 - AWS
- Google Cloud
- Azure
- Oracle
- Or simulate the Cloud locally on your computer.

 Possible backend technologies include Firebase, MQTT, InfluxDB, and AWS Amplify.

 ## 3\. Minimum required tasks

 Your project must cover these core tasks:

 1. **Program the PCB** and collect sensor data.
2. **Transmit the data via BLE.**
3. **Receive BLE data** on Android or another Edge device.
4. **Send data from Edge to Cloud.**
5. **Store data** on the Cloud/server.
6. Create **an ML model** at Edge and/or Cloud level.
7. **Display the data to users**, for example through a web dashboard.

 The ML component is explicitly included in the task list, so it should be part of your project design.

 ## 4. Reports and evaluation

 The final grade is:

 | Component | Weight |
| --- | --- |
| 1st report | **20%** |
| 2nd report | **20%** |
| Final project | **60%** |
| **Total** | **100%** |

### First report — BLE

 The first report concerns the **PCB + Android/Edge BLE communication**.

 **Deadline: 30 October**

 You submit it through the Virtual Campus.

 ### Final project evaluation

 **7 January**

 The project is evaluated in the lab. The guidelines note that any date change will be announced through the Virtual Campus.

 ### Second report

 **10 January**

 Submit:

 - Backend
- Frontend
- Project code as a ZIP **or GitHub repository**

 through the Virtual Campus.

 ## 5\. Required report structure

 Both reports should be written as **professional technical documentation**, treating your group as a company delivering an IoT product.

 ### 1\. Executive Summary & Project Overview

 Explain:

 - Your fictional/company identity.
- The IoT product.
- Target use case.
- Project goals.
- Main system capabilities.

 ### 2\. System Architecture

 Include a **complete block diagram** containing:

 **Sensors/PCB → Edge/Mobile/Gateway → Cloud**

 Show:

 - Interfaces.
- Communication channels.
- Data flows.

 ### 3\. Implementation Details

 Document each layer.

 **Device:**

 - Board/microcontroller.
- Sensors.
- Firmware framework.
- Interrupts.
- Power management.

 **Edge:**

 - Android/Raspberry Pi/laptop.
- OS version.
- BLE scanning.
- BLE connection management.

 **Cloud:**

 - Backend.
- Database type/strategy.
- Frontend/web UI.
- ML model and its purpose.

 Also provide a list of **all third-party libraries, APIs, and frameworks** used.

 ## 6\. You must document your communication protocols

 This is particularly important.

 ### BLE data frame

 You must explicitly define the structure of the data sent from the PCB.

 For example:

```
[Header][Sensor 1][Sensor 2][Timestamp][CRC]
```

 The actual structure is up to your group, but the report must explain the **exact byte layout**.

 ### Edge → Cloud format

 Document:

 - Communication protocol: MQTT, HTTP REST, WebSockets, etc.
- JSON/payload structure.
- Fields and their meanings.

 For example:

```
{
  "device_id": "device01",
  "timestamp": "...",
  "temperature": 23.5,
  "humidity": 51.2
}
```

 The important point is that **your group's communication strategy must be explicitly specified and documented**.

 ## 7\. Testing and validation

 You need evidence that the system actually works.

 ### Per-layer testing

 Test individual components, such as:

 - Sensor-reading accuracy.
- PCB functionality.
- BLE connection stability.
- API responses.
- Database operations.

 ### End-to-end testing

 Demonstrate the complete path:

 **Physical measurement → PCB → BLE → Edge → Cloud → Database → Dashboard**

 ### Evidence

 Include things such as:

 - Graphs.
- Logs.
- Screenshots.
- Test results.

 Don't just say that something works—provide evidence.

 ## 8\. Team and individual assessment

 The **second report** must include a final section about your team and the course.

 ### Team responsibilities

 Include a **task distribution matrix**, showing who worked on:

 - Firmware/PCB
- Android/Edge
- Backend
- Frontend
- ML
- Other components

 Each member needs to make a meaningful contribution.

 ### Self-assessment

 Each student should discuss:

 - Technical development.
- Problem-solving.
- Personal contribution.
- Commitment to the project.

 ### Team collaboration

 Explain:

 - Version control.
- Task management.
- Meeting frequency.
- How decisions were made.
- How technical/interpersonal problems were resolved.

 ## 9\. Course feedback

 The second report should also provide constructive feedback.

 Discuss:

 **Strengths**

 - What helped your learning?
- Which labs/tools/project aspects were useful?

 **Areas for improvement**

 - Instructions.
- Framework recommendations.
- Hardware resources.
- Schedule.
- Evaluation criteria.
- Anything else that could improve the course.

 ## 10\. The essential checklist

 If you want to make sure your group satisfies the requirements, the core checklist is:

 - [ ] Group of 3
- [ ] Define an IoT use case/product
- [ ] Program nRF52840-DK
- [ ] Connect sensors
- [ ] Send sensor data using BLE
- [ ] Build/configure Edge device
- [ ] Scan/connect/discover BLE services
- [ ] Receive BLE notifications
- [ ] Send Edge data to Cloud
- [ ] Store data in a database
- [ ] Build a user-facing visualization/dashboard
- [ ] Implement an ML model for prediction/detection
- [ ] Define BLE packet format
- [ ] Define Edge-to-Cloud protocol/payload
- [ ] Test each layer
- [ ] Perform end-to-end testing
- [ ] Collect screenshots/logs/graphs as evidence
- [ ] Submit **Report 1 by October 30**
- [ ] Demonstrate project **January 7**
- [ ] Submit **Report 2 + code by January 10**
- [ ] Include team contribution/self-assessment/course feedback

 ### In one sentence

 **The lab is essentially asking you to build and document a complete IoT pipeline—sensor/PCB → BLE → Edge → Cloud/database → visualization \+ ML—and prove through testing and professional reports that every part works.**
