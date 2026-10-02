```
Bus 001 Device 002: ID 1366:1025 SEGGER J-Link
```

 That confirms:

```
nRF52840-DK
     │
     │ USB
     ▼
Windows 10
     │
     │ usbipd-win
     ▼
WSL2
     │
     │ USB/IP
     ▼
SEGGER J-Link
```

 The **SEGGER J-Link** is the onboard debug/programming interface of the Nordic development kit. It is what we'll use to program and debug the nRF52840.

 ## Phase 0 milestone #1: COMPLETE ✅

 We have now established:

 - Windows 10 → ✅
- WSL2 → ✅
- Ubuntu → ✅
- Python → ✅
- Git → ✅
- `usbutils` → ✅
- `usbipd-win` → ✅
- Windows → WSL USB passthrough → ✅
- nRF52840-DK visible inside Linux → **✅**

 And importantly, we have **not yet installed the embedded development stack**, so the environment is still clean.

 ### One small housekeeping point

 You're again starting in:

```
/mnt/c/Users/louis
```

 Before we start installing Zephyr, let's move to your Linux home:

```
cd ~
pwd
```

 We should get:

```
/home/louis
```

 From now on, we'll do the development work there.

---

 ## Next phase: Zephyr installation

 Now we can move to the actual Nordic development environment:

```
WSL
 │
 ├── Python environment
 ├── west
 ├── Zephyr
 ├── Zephyr SDK
 ├── CMake
 ├── Ninja
 └── nRF52840 toolchain
          │
          ▼
      nRF52840-DK
```

 **Before installing Zephyr, however, I want to verify the current Zephyr-supported Python versions and installation procedure**, because your Ubuntu has **Python 3.14.4**, which is newer than the Python version commonly used by older Zephyr releases. We should avoid creating an environment around an incompatible Python version.

 The next step should therefore be **checking the current Zephyr requirements**, then installing the exact prerequisites rather than blindly following an older tutorial.
