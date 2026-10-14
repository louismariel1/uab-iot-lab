SSP IoT POC — Project Bootstrap and Development Handover

Project: SSP IoT Proof of Concept
Repository: ssp-iot-poc
Development environment: Windows + VS Code + WSL2 (Ubuntu)
Firmware framework: Zephyr RTOS 3.12
Python tooling: Python virtual environment .venv-ble
Current Git branch: main
Document purpose: Record the current implementation status, architectural decisions, outstanding work, and agreed development workflow.

1. Project goal

Build an IoT proof of concept that demonstrates BLE-based communication between a Zephyr-powered peripheral and an Android Edge application.

The intended system includes:

A Zephyr firmware application that exposes an SSP BLE GATT service.
A shared, versioned BLE protocol definition maintained in YAML.
Generated C protocol definitions and payload encoding for firmware.
Generated Kotlin protocol definitions and payload decoding for Android.
Synthetic motion-event generation and BLE notification delivery.
An Android application that scans for, connects to, and monitors the peripheral.
An Android state machine that processes received events and records monitoring state.
Shared protocol test vectors and automated tests.
Documentation generated from the authoritative protocol definition.

The initial objective is a functional, reproducible demonstration using synthetic events. Real sensor integration, production security, and broader system integration can follow after the communication path works.

2. Architectural overview
                 Authoritative protocol
                  protocol/ssp_ble_v1.yaml
                            |
                            v
                    Python code generator
                    tools/ble_codegen/
                            |
                 +----------+----------+
                 |          |          |
                 v          v          v
             C header    Kotlin API   Markdown
                 |          |          |
                 v          v          v
           Zephyr BLE    Android     Protocol
           peripheral     Edge       reference
                 |          ^
                 |          |
                 +--- BLE --+
                   GATT / Notify

             Shared test vectors
              tests/ble/


The YAML file is the authoritative source for protocol constants, UUIDs, payload layout, field semantics, demo configuration, and test vectors.

Generated files should be regenerated from the YAML rather than manually maintained independently.

3. Repository structure

The repository currently contains the following main areas:

ssp-iot-poc/
├── cloud/
│   ├── backend/
│   ├── database/
│   └── frontend/
├── device/
│   ├── drivers/
│   ├── firmware/
│   │   ├── hello_world/
│   │   └── ble_peripheral/
│   │       ├── CMakeLists.txt
│   │       ├── Kconfig
│   │       ├── prj.conf
│   │       └── src/
│   │           ├── main.c
│   │           ├── ssp_advertising.c
│   │           ├── ssp_advertising.h
│   │           ├── ssp_event.c
│   │           ├── ssp_event.h
│   │           ├── ssp_gatt.c
│   │           ├── ssp_gatt.h
│   │           └── generated/
│   │               └── ssp_protocol.h
│   ├── protocol/
│   └── tests/
├── docs/
│   ├── architecture/
│   ├── protocol/
│   │   └── ssp-ble-v1.md
│   ├── reports/
│   └── testing/
├── edge/
│   ├── android/
│   │   ├── settings.gradle.kts
│   │   ├── build.gradle.kts
│   │   ├── gradlew
│   │   └── app/
│   │       └── src/
│   │           ├── main/
│   │           │   ├── AndroidManifest.xml
│   │           │   └── java/org/ssp/
│   │           │       ├── protocol/SspProtocol.kt
│   │           │       ├── state/SspState.kt
│   │           │       ├── ble/SspBleClient.kt
│   │           │       ├── ble/AndroidSspBleClient.kt
│   │           │       └── ui/
│   │           └── test/java/org/ssp/
│   │               ├── protocol/SspProtocolTest.kt
│   │               └── state/SspStateTest.kt
│   ├── protocol/
│   └── tests/
├── ml/
├── protocol/
│   └── ssp_ble_v1.yaml
├── tests/
│   └── ble/
│       ├── test_vectors.json
│       └── test_protocol.py
├── tools/
│   ├── ble_codegen/
│   │   ├── requirements.txt
│   │   ├── generate.py
│   │   └── templates/
│   │       ├── ssp_protocol.h.j2
│   │       ├── SspProtocol.kt.j2
│   │       └── protocol.md.j2
│   ├── create_ble_protocol_structure.sh
│   ├── create_zephyr_ble_structure.sh
│   ├── create_android_edge_structure.sh
│   ├── list_empty_files.sh
│   ├── compile_and_flash_test.sh
│   ├── flash_and_uart_smoke_test.sh
│   └── verification.sh
└── PROJECT_BOOTSTRAP.md


This is a logical inventory based on the development history, not a fresh filesystem verification. The existence and current contents of individual files should be confirmed before relying on this tree.

4. Authoritative BLE protocol

Source file: protocol/ssp_ble_v1.yaml

The protocol currently defines:

Property	Value
Protocol name	SSP BLE Protocol
Version	1
Device name	SSP-POC
Device ID	0x53535001
Byte order	Little-endian
Payload name	SSPEventV1
Payload length	17 bytes
Configured maximum payload	20 bytes
Service	7d2ea28a-f7bd-485a-bd9d-92ad6ecfe93e
Status characteristic	7d2ea28a-f7bd-485a-bd9d-92ad6ecfe940
Event characteristic	7d2ea28a-f7bd-485a-bd9d-92ad6ecfe941
Payload layout
Field	Type	Size	Offset
version	u8	1	0
event_type	u8	1	1
status	u8	1	2
device_id	u32	4	3
sequence	u32	4	7
timestamp_ms	u32	4	11
motion_mg	u16	2	15
Total		17	

Defined event values:

NONE = 0
MOTION_EVENT = 1

Defined status values:

OK = 0
SENSOR_ERROR = 1

The current synthetic test event uses:

Event type: MOTION_EVENT
Status: OK
Sequence: 1
Timestamp: 1000 milliseconds
Motion metric: 1800 milli-g

Expected payload:

0101000150535301000000e80300000807


The device ID must remain consistent with the YAML definition. Its decimal value is 1397968897.

Protocol behavior still requiring decisions
The state transition from NORMAL to ELEVATED is defined.
The return-to-NORMAL behavior is not yet specified.
The motion metric is provisional and belongs to the synthetic demonstration.
The event notification behavior must be implemented and verified on both sides.
GATT status read behavior must be explicitly defined and implemented.
The permissions: [none] convention for the notification characteristic must be handled consistently by the generator and firmware implementation.
5. Python code generation

Relevant files:

tools/ble_codegen/requirements.txt
tools/ble_codegen/generate.py
tools/ble_codegen/templates/ssp_protocol.h.j2
tools/ble_codegen/templates/SspProtocol.kt.j2
tools/ble_codegen/templates/protocol.md.j2

The generator validates the YAML protocol definition, checks field widths and payload length, validates UUIDs and shared test vectors, and renders the generated C, Kotlin, and Markdown files.

The generator's validation-only mode is:

python tools/ble_codegen/generate.py --check


Generation mode is:

python tools/ble_codegen/generate.py


Generation produces:

device/firmware/ble_peripheral/src/generated/ssp_protocol.h
edge/android/app/src/main/java/org/ssp/protocol/SspProtocol.kt
docs/protocol/ssp-ble-v1.md
tests/ble/test_vectors.json
Python environment

The dedicated virtual environment is:

.venv-ble/


It was created separately from the Zephyr Python environment to avoid modifying Zephyr's dependencies.

The following packages have been installed in that environment:

PyYAML 6.0.3
Jinja2 3.1.6
MarkupSafe, installed as a Jinja2 dependency

Activate the environment from the repository root:

source .venv-ble/bin/activate


Install generator dependencies if needed:

python -m pip install -r tools/ble_codegen/requirements.txt


Do not install project-specific generator dependencies into the system Python or the existing Zephyr environment unless there is a specific reason to do so.

Previously completed validation

The following checks previously succeeded:

python -m py_compile tools/ble_codegen/generate.py
python tools/ble_codegen/generate.py --check


The shared Python protocol tests also previously passed:

python -m unittest discover -s tests/ble -p 'test_*.py' -v

Ran 3 tests
OK


These results are historical. They do not establish that the current working tree still passes after subsequent edits.

6. Zephyr BLE peripheral status

The peripheral project structure has been created and committed.

Relevant files:

device/firmware/ble_peripheral/CMakeLists.txt
device/firmware/ble_peripheral/prj.conf
device/firmware/ble_peripheral/Kconfig
device/firmware/ble_peripheral/src/main.c
device/firmware/ble_peripheral/src/ssp_gatt.c
device/firmware/ble_peripheral/src/ssp_gatt.h
device/firmware/ble_peripheral/src/ssp_event.c
device/firmware/ble_peripheral/src/ssp_event.h
device/firmware/ble_peripheral/src/ssp_advertising.c
device/firmware/ble_peripheral/src/ssp_advertising.h
Implemented or proposed at the source level
Firmware build configuration.
Bluetooth peripheral configuration.
Application entry point.
Event producer interface.
Advertising module and header.
GATT module interface and implementation file location.
Generated protocol header with explicit little-endian event serialization.
Important gaps

The GATT source file was reported as empty in the most recent empty-file inventory. It must be implemented before treating the peripheral as functional.

The firmware must correctly:

Initialize Bluetooth.
Start connectable advertising with the intended device name and service UUID.
Expose the status characteristic and its read handler.
Expose the event characteristic with the correct notification configuration.
Track the subscribed connection.
Generate and encode synthetic events.
Notify the subscribed connection using the correct characteristic attribute.
Handle disconnects and release connection references.
Stop or suspend event delivery when notifications are not enabled.

The firmware has not yet been confirmed to compile, flash, advertise, or communicate with an Android client.

7. Android Edge application status

The Android directory structure has been created, but the Android application is not yet a confirmed buildable project.

Relevant files include:

edge/android/settings.gradle.kts
edge/android/build.gradle.kts
edge/android/gradlew
edge/android/app/build.gradle.kts
edge/android/app/src/main/AndroidManifest.xml
edge/android/app/src/main/java/org/ssp/protocol/SspProtocol.kt
edge/android/app/src/main/java/org/ssp/state/SspState.kt
edge/android/app/src/main/java/org/ssp/ble/SspBleClient.kt
edge/android/app/src/main/java/org/ssp/ble/AndroidSspBleClient.kt
Protocol and state-machine unit-test files.

The shared Kotlin protocol decoder was generated from the YAML definition.

State machine

The proposed Android BLE client must implement the following lifecycle:

IDLE
  |
  v
SCANNING
  |
  v
CONNECTING
  |
  v
DISCOVERING_SERVICES
  |
  v
ENABLING_NOTIFICATIONS
  |
  v
READY
  |
  v
DECODE -> VALIDATE -> UPDATE STATE


The implementation must verify the SSP service and event characteristic before subscribing. It must wait for successful CCCD descriptor-write completion before entering READY.

It must also handle failures, timeouts, disconnections, permissions, and cleanup.

State management

SspState.kt has been proposed with:

NORMAL and ELEVATED monitoring states.
Protocol version and status validation.
Motion-event validation.
Duplicate and out-of-order sequence rejection.
Unsigned 32-bit sequence rollover handling.
Reset support for a new session.

The source has not yet been confirmed to compile or pass Android unit tests.

BLE interface

SspBleClient.kt has been proposed as a platform-independent interface using callbacks for:

Connection changes.
Event delivery.
Error reporting.

Device discovery must also be exposed so the UI can present scan results and allow the user to select a peripheral.

Android build status

The following files were listed as zero-byte placeholders during the most recent inventory:

edge/android/settings.gradle.kts
edge/android/build.gradle.kts
edge/android/app/build.gradle.kts
edge/android/app/src/main/AndroidManifest.xml
edge/android/gradlew
edge/android/app/src/main/java/org/ssp/ble/AndroidSspBleClient.kt
edge/android/app/src/test/java/org/ssp/protocol/SspProtocolTest.kt
edge/android/app/src/test/java/org/ssp/state/SspStateTest.kt

The Android build configuration and tests therefore remain immediate work items. The Gradle wrapper script is only a placeholder; it must be generated as a real wrapper using a compatible Gradle installation and project configuration.

8. Empty-file inventory tooling

The following script was created:

tools/list_empty_files.sh

It lists zero-byte files while excluding common generated directories and virtual environments.

Run it from the repository root:

bash tools/list_empty_files.sh


The most recent run reported 11 zero-byte files:

device/firmware/ble_peripheral/Kconfig
device/firmware/ble_peripheral/src/ssp_gatt.c
edge/android/gradlew
edge/android/app/src/main/AndroidManifest.xml
edge/android/app/src/main/java/org/ssp/ui/.gitkeep
edge/android/app/src/main/java/org/ssp/ble/AndroidSspBleClient.kt
edge/android/app/src/test/java/org/ssp/protocol/SspProtocolTest.kt
edge/android/app/src/test/java/org/ssp/state/SspStateTest.kt
edge/android/app/build.gradle.kts
edge/android/build.gradle.kts
edge/android/settings.gradle.kts

This inventory should be refreshed before using it as a current status report.

9. Agreed development workflow

Development is incremental and file-oriented.

For each proposed file:

Review its content and validate its design.
Correct errors or inconsistencies before implementation.
Include the intended repository path as the first-line comment where the file format supports comments.
Present the complete approved file content.
The developer pastes the content into the repository.
Continue to the next file or dependent component.

Do not create unnecessary scripts or diagnostic steps for ordinary source-file edits.

Run checks when they provide meaningful evidence, such as when establishing the build system, validating generated outputs, or confirming that integrated functionality compiles and runs.

An empty placeholder should not be populated merely to eliminate zero-byte files. Populate it when the next implementation step requires it.

Do not claim that source code has been compiled, flashed, or tested unless that verification actually occurred.

10. Immediate next step

Implement the Android Gradle project configuration.

Recommended order:

edge/android/settings.gradle.kts
edge/android/build.gradle.kts
edge/android/app/build.gradle.kts
edge/android/app/src/main/AndroidManifest.xml

Choose compatible Android Gradle Plugin, Kotlin, and Gradle versions and document the selected versions. Configure the SDK and test dependencies without introducing unnecessary libraries.

Then generate the real Gradle wrapper.

Once the Android project configuration is in place:

Add SspProtocolTest.kt.
Add SspStateTest.kt.
Finalize the SspBleClient interface, including device discovery and lifecycle state.
Implement AndroidSspBleClient.kt.
Implement the UI and event log.
Implement the Zephyr GATT module and connect it to advertising and synthetic event generation.
11. Planned integration and verification

The eventual end-to-end demonstration should verify the following sequence:

Build the Zephyr peripheral firmware.
Flash the target device.
Confirm that the device advertises with the SSP identity and service UUID.
Launch the Android application and scan for the peripheral.
Connect and verify the SSP GATT service.
Enable notifications and wait for CCCD write success.
Receive the synthetic event.
Decode the 17-byte payload.
Validate the protocol version, event type, status, device ID, and sequence.
Update the monitoring state to ELEVATED.
Display the event in the Android monitoring UI.
Confirm disconnect and reconnect behavior.
Verify shared test vectors and automated tests.

The initial milestone is a reliable synthetic-event demonstration. Hardware sensor integration and further production hardening should be considered after that milestone succeeds.

12. Known limitations and decisions still open
The Android application has not been confirmed to build.
The Zephyr BLE peripheral has not been confirmed to compile or run end to end.
The GATT implementation needs completion.
The Android BLE state machine needs implementation.
The monitoring UI and event log remain to be developed.
The behavior for returning from ELEVATED to NORMAL is unspecified.
Production BLE security, bonding, reconnection policy, and device identity verification need explicit decisions before production use.
Protocol and firmware configuration must remain aligned with the YAML source.
Existing source files and Git status should be checked before committing further work.
13. Instructions for the next development session

Start by reading this bootstrap document and inspecting the actual repository state.

Do not assume every file listed above has the described content. Some entries reflect proposed implementations, while others reflect previously observed files.

Resume at Android Gradle project configuration, beginning with:

edge/android/settings.gradle.kts

Continue the agreed workflow: validate one file at a time, return complete corrected content, and distinguish proposed code from verified functionality.
