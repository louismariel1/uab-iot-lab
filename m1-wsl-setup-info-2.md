That is useful context. Since USB access from WSL previously worked on this laptop, we can treat **WSL2 USB access as a feasible path**, but still verify it during setup rather than assuming the previous configuration survived the reinstallation.

 Below is the complete **Phase 0 installation sequence**, deliberately ordered so that we establish the foundation before adding SSP-specific software.

 ## Phase 0 — Complete development-environment setup

 ### Target architecture

```
┌────────────────────────────── WINDOWS 10 ──────────────────────────────┐
│                                                                       │
│  VS Code ───────────────┐                                             │
│                         │                                             │
│  usbipd-win              │                                             │
│       │                  │                                             │
│       │ USB              ▼                                             │
│       │             ┌──────────── WSL2 / Ubuntu ──────────────┐       │
│       │             │                                          │       │
│       └────────────►│ Git                                      │       │
│                     │ Python + virtual environments             │       │
│                     │ C/C++ toolchain                           │       │
│                     │ CMake + Ninja                             │       │
│                     │ west                                      │       │
│                     │ Zephyr                                    │       │
│                     │ Zephyr SDK                                │       │
│                     │ Nordic tools                              │       │
│                     │ SSP Python environment                     │       │
│                     └─────────────────┬────────────────────────┘       │
│                                       │                                │
└───────────────────────────────────────┼────────────────────────────────┘
                                        │ USB
                                        ▼
                              ┌──────────────────┐
                              │ nRF52840-DK      │
                              │                  │
                              │ Zephyr firmware  │
                              │ Sensors / GNSS   │
                              │ BLE              │
                              └──────────────────┘
```

 The important principle is:

 > **Windows is the host; WSL2 is our primary development environment; the Nordic board is the target device.**

---

 # Installation sequence

 ## 1\. Verify Windows 10 baseline

 **Install:** Nothing initially.

 **Purpose:** Make sure the reinstalled laptop has the required Windows/WSL foundation.

 Check:

```
winver
wsl --status
wsl --list --verbose
```

 We want:

```
Windows 10
WSL 2
Ubuntu
```

 If WSL isn't properly configured, fix this before proceeding.

---

 ## 2\. Update WSL2

 **Component:** WSL2

 **Role:** Provides our Linux development environment while keeping Windows as the host OS.

 Run from PowerShell:

```
wsl --update
wsl --shutdown
```

 Then verify the distribution is actually running under WSL2.

---

 ## 3\. Prepare Ubuntu/WSL

 Inside Ubuntu:

```
sudo apt update
sudo apt upgrade
```

 **Role:** Provides the Linux environment in which we'll run Zephyr, Python, Git, compilers, build tools, etc.

 I'd keep essentially all source code under the Linux filesystem, for example:

```
~/projects/
```

 rather than developing directly under `/mnt/c/...`. This generally gives Linux development tools much better filesystem performance.

---

 # 4\. Install Git

 **Component:** Git

 **Role:** Version control for the entire SSP project.

 We'll eventually have:

```
ssp-poc/
├── device/
├── edge/
├── backend/
├── ml/
├── simulation/
├── tests/
└── docs/
```

 Git allows us to:

 - track changes,
- create branches,
- recover previous versions,
- collaborate,
- eventually use GitHub,
- document exactly which firmware/software version produced a result.

---

 # 5\. Install Python 3.12

 **Component:** Python

 **Role:** Python will be our main language for the non-embedded parts of the PoC.

 Potential uses:

 - backend services,
- data processing,
- simulation,
- test automation,
- ML training,
- ML inference,
- data analysis,
- KPI calculations.

 **Important:** Python will **not** replace the Zephyr firmware on the nRF52840.

 The architecture will eventually be:

```
nRF52840 firmware → C / Zephyr

Edge/backend/ML   → Python where appropriate
```

---

 # 6\. Install Python virtual-environment support

 **Component:** `venv`

 **Role:** Prevents different projects and tools from contaminating each other's Python dependencies.

 We'll eventually have something like:

```
~/zephyrproject/
    .venv/

~/projects/ssp-poc/
    .venv/
```

 The Zephyr environment and SSP application environment should remain separate.

---

 # 7\. Install basic Linux development dependencies

 Install the standard development foundation:

 - GCC/G++
- CMake
- Ninja
- Make
- `gperf`
- Device Tree Compiler
- Python development packages
- `ccache`
- other Zephyr prerequisites

 **Role:** These are the underlying tools used to compile and build the embedded firmware.

 Conceptually:

```
Zephyr source
     ↓
C/C++ compiler
     ↓
CMake/Ninja
     ↓
firmware binary
     ↓
nRF52840
```

---

 # 8\. Install `west`

 **Component:** `west`

 **Role:** Zephyr's project/workspace management tool.

 It will become one of our main commands.

 For example:

```
west init
west update
west build
west flash
```

 Think of it as the central command-line tool for managing the Zephyr development workspace.

---

 # 9\. Create the Zephyr workspace

 Create:

```
~/zephyrproject/
```

 This becomes our embedded-development workspace.

 Conceptually:

```
zephyrproject/
├── .west/
├── zephyr/
├── modules/
├── tools/
└── .venv/
```

 **Role:** Contains Zephyr itself and the modules required to build firmware for the nRF52840.

---

 # 10\. Install the Zephyr SDK

 **Component:** Zephyr SDK

 **Role:** Provides the compiler/toolchain and related tools required to build Zephyr applications for the target processor.

 This is distinct from Zephyr itself.

 Think:

```
Zephyr
   +
Zephyr SDK
   ↓
Firmware for nRF52840
```

---

 # 11\. Install/configure VS Code integration

 You already have VS Code, so we don't need another editor.

 Install/configure the relevant extensions, particularly:

 - WSL/Remote Development
- C/C++
- Python
- Git
- Nordic nRF Connect for VS Code
- Zephyr-related support as appropriate

 **Role:** Gives us the graphical development environment.

 The desired experience is:

```
Windows VS Code
       ↓
WSL environment
       ↓
Zephyr project
       ↓
nRF52840
```

 You should be able to open the project in VS Code and have the actual compiler/toolchain running inside WSL.

---

 # 12\. Install `usbipd-win` on Windows

 **Component:** `usbipd-win`

 **Important:** This is installed on **Windows**, not inside WSL.

 **Role:** Bridges physical USB devices from Windows into WSL2.

 Our Nordic board is physically connected to:

```
Laptop USB
     ↓
Windows
     ↓
usbipd-win
     ↓
WSL2
```

 Then Linux should be able to see the board.

---

 # 13\. Test USB passthrough with the nRF52840-DK

 Connect:

```
nRF52840-DK
      │
     USB
      │
      ▼
Windows laptop
```

 Then expose the relevant USB device to WSL.

 Inside WSL we'll verify it with:

```
lsusb
```

 **Role:** This is the critical hardware integration test.

 At the end of this step we want:

```
Windows sees Nordic board
             +
WSL sees Nordic board
```

 Don't connect the GNSS module yet.

---

 # 14\. Install/configure Nordic development tooling

 **Component:** Nordic tooling / nRF Connect ecosystem

 **Role:** Provides Nordic-specific support for the nRF52840 and helps with:

 - board identification,
- flashing,
- debugging,
- device management,
- development.

 We don't need to turn this into a huge collection of Nordic applications. We'll install only what the actual Zephyr/nRF52840 workflow requires.

---

 # 15\. Verify the nRF52840-DK itself

 Before writing SSP software, establish:

```
WSL
 ↓
Zephyr
 ↓
nRF52840-DK
```

 We should be able to identify the board and target it as:

```
nrf52840dk/nrf52840
```

 **Role:** Confirms that our development environment recognizes the actual hardware we're going to build the PoC around.

---

 # 16\. Build and flash Zephyr "Hello World"/Blinky

 This is our **first real hardware milestone**.

```
Zephyr example
      ↓
     build
      ↓
    flash
      ↓
nRF52840-DK
      ↓
LED / console
```

 **Role:** Proves the complete toolchain:

```
Python
 ↓
west
 ↓
CMake/Ninja
 ↓
compiler
 ↓
Zephyr
 ↓
USB
 ↓
nRF52840
```

 If this works, we know the basic development environment is sound.

---

 # 17\. Verify serial/console communication

 Next:

```
nRF52840
    ↓
serial output
    ↓
WSL
```

 We'll eventually have firmware saying something like:

```
SSP Device starting
Firmware version: 0.1.0
BLE initialized
```

 **Role:** Gives us a basic diagnostic channel that will be invaluable during the PoC.

---

 # 18\. Install a BLE testing tool

 For the initial BLE test, I recommend using a phone with **nRF Connect** rather than immediately writing our own Edge application.

 **Role:** It gives us a known-good BLE client for testing the Device layer.

 The progression becomes:

```
nRF52840
    ↓
BLE advertisement
    ↓
nRF Connect
    ↓
Device visible
```

 Then:

```
nRF52840
    ↓
BLE GATT service
    ↓
nRF Connect
    ↓
Characteristic visible
```

 This isolates BLE from the rest of the SSP application.

---

 # 19\. Create the SSP Git repository

 Only once the environment works, create the actual project structure:

```
ssp-poc/
│
├── device/
│   └── firmware/
│
├── edge/
│
├── backend/
│
├── ml/
│
├── simulation/
│
├── tests/
│
├── data/
│
├── docs/
│
└── README.md
```

 **Role:** Establishes the project as an engineering product rather than a collection of experiments.

---

 # 20\. Create the SSP Python environment

 Now create:

```
ssp-poc/.venv/
```

 This will eventually contain packages for things such as:

```
FastAPI
MQTT client
NumPy
Pandas
scikit-learn
PyTest
etc.
```

 **But don't install all of those yet.**

 We'll install dependencies when each layer actually requires them.

---

 # 21\. Install Python development tools

 Once the basic environment works, we'll add:

 - PyTest — automated testing
- formatting/linting tools
- data analysis tools
- backend framework
- MQTT/HTTP libraries
- ML libraries

 **Role:** These form the Python side of the Edge/Cloud/ML development.

---

 # 22\. Android development environment — later in Phase 0/1

 We don't need Android Studio to establish the Nordic development environment.

 But because the laboratory explicitly allows/recommends Android as the Edge, we'll eventually install:

 - Android Studio
- Android SDK
- Android emulator/tools
- Kotlin support

 **Role:**

```
nRF52840
     ↓ BLE
Android phone
     ↓
Internet
     ↓
Cloud
```

 This is likely to become our **final SSP Edge architecture**.

---

 # 23\. Cloud/backend environment — after Device + BLE

 Only after the Device/BLE foundation works should we install/configure:

 - MQTT or HTTP,
- backend,
- database,
- dashboard,
- ML environment.

 The eventual architecture becomes:

```
              SSP PoC

       DEVICE
          │
       Sensors
          │
       Zephyr
          │
         BLE
          │
          ▼
        EDGE
    Android/Kotlin
          │
       MQTT/HTTP
          │
          ▼
        CLOUD
    ┌─────┼─────┐
    │     │     │
 Backend DB    ML
    │     │     │
    └─────┼─────┘
          │
      Dashboard
```

---

 # The complete dependency order

 If we compress everything into a single checklist:

```
WINDOWS HOST
│
├─ 1. Verify Windows 10
├─ 2. Verify/update WSL2
├─ 3. Verify Ubuntu
├─ 4. Install usbipd-win
│
└── WSL / UBUNTU
    │
    ├─ 5. Update Ubuntu
    ├─ 6. Git
    ├─ 7. Python 3.12
    ├─ 8. Python venv
    ├─ 9. GCC/G++
    ├─ 10. CMake
    ├─ 11. Ninja
    ├─ 12. Zephyr prerequisites
    ├─ 13. west
    ├─ 14. Zephyr workspace
    ├─ 15. Zephyr SDK
    │
    ├─ 16. VS Code ↔ WSL
    ├─ 17. Nordic VS Code tooling
    │
    ├─ 18. USB → WSL test
    ├─ 19. nRF52840 detection
    ├─ 20. Zephyr Blinky
    ├─ 21. Serial console
    ├─ 22. BLE beacon
    └─ 23. BLE GATT test
         │
         ▼
    SSP PROJECT
    │
    ├─ 24. Git repository
    ├─ 25. Python .venv
    ├─ 26. Python tooling
    │
    ├─ 27. Android Studio
    ├─ 28. Android/Kotlin BLE
    │
    ├─ 29. Backend
    ├─ 30. Database
    ├─ 31. MQTT/HTTP
    ├─ 32. Dashboard
    └─ 33. ML environment
```

 ## What we should _not_ install yet

 To keep Phase 0 controlled, I would **not yet install**:

 - Arduino IDE
- PlatformIO
- Raspberry Pi software
- TensorFlow/PyTorch
- database servers
- MQTT brokers
- cloud SDKs
- GPS/GNSS libraries
- a collection of random Python packages

 The lab gives us several options, but we don't need all of them. **Zephyr + nRF52840 + BLE is our Device foundation.**

 ### Our Phase-0 definition of "ready"

 I would consider the environment ready when we can demonstrate:

 > **WSL → Zephyr → build → USB → nRF52840-DK → firmware runs → serial output works → BLE advertisement is visible.**

 At that point we have a solid foundation for building the actual **SSP PoC**, rather than debugging the development environment and the application simultaneously.

 **Recommended next action:** don't install anything blindly yet. We should first inspect the freshly reinstalled machine with about 6 commands, determine exactly what is already present, and then install only what's missing in the sequence above.
