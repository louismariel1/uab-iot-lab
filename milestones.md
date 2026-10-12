## TD1 — Create GitHub milestone definitions

The 21 roadmap phases (0–20) should be consolidated into 11 GitHub milestones, M0–M10, as proposed in the roadmap. A milestone can contain several related phases, issues, and deliverables.

| GitHub milestone | Scope | Completion criterion |
| --- | --- | --- |
| M0 — Development Environment | Roadmap Phase 0 | Firmware, Android and backend development environments are reproducible. |
| M1 — SSP Use Case and Requirements | Phase 1 | Use case, system architecture, events, risk model and KPIs are defined. |
| M2 — Sensor Integration | Phase 2 | Sensors produce reliable, timestamped measurements on the nRF52840. |
| M3 — Device Firmware and BLE Protocol | Phases 3–5 | Data model and BLE protocol are specified, and the device transmits valid SSP data. |
| M4 — Android BLE Integration | Phases 6–7 | Android receives and displays live device measurements. |
| M5 — Edge Processing and Risk | Phases 8–9 | Edge validation, feature extraction, event detection and risk classification work. |
| M6 — Adaptive Monitoring and Edge ML | Phases 10–11 | Adaptive policies and ML inference are implemented and evaluated. |
| M7 — Cloud Backend and Database | Phases 12–13 | Edge data reaches the cloud and is persistently stored. |
| M8 — Dashboard and Alerts | Phase 14 | Users can view device state, historical data, events and alerts. |
| M9 — End-to-End Validation and KPIs | Phases 15–18 | Security, privacy, performance and failure scenarios are evaluated. |
| M10 — Final Integration and Documentation | Phases 19–20 | A clean checkout supports reproducible deployment, demonstration and reporting. |

The milestone boundaries are provisional. In particular, M3 and M4 may need adjustment once the actual sensor, firmware and Android integration work is fully understood.

## TD2 — Finalize M2 from the existing project documentation

We will use the documents you submit as evidence of the project's actual state, rather than assuming that every original roadmap activity remains appropriate.

For M2, I suggest we establish the following:

1. Current state: what has already been implemented and tested.
2. Remaining scope: what M2 must deliver before it can close.
3. Technical interfaces: hardware, firmware, sensor drivers, configuration and data output.
4. Acceptance criteria: objective tests that establish completion.
5. GitHub issues: actionable tasks linked to the milestone.
6. Evidence and documentation: build logs, test results, wiring information and reproducible commands, where applicable.

We should also distinguish between work that is already complete, work that has been attempted but is not yet verified, and work that has not started.

The recent PCB smoke test is a useful example of the evidence standard: the nRF52840 firmware was flashed successfully, and the UART listener received the expected `SSP PoC Hello World! nRF52840` message. The automated test passed after correcting the expected UART string. That verifies the current flash-and-UART path, but does not by itself establish that sensor integration is complete.

Next step: submit the current M2 documents. I will use them to build a consolidated M2 definition before we finalize its GitHub milestone and associated issues. We will defer roadmap changes until that review is complete.
