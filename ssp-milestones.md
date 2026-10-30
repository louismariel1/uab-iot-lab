This is the clearest SSP project view I can derive from everything we've discussed so far, including:

Official IoT Lab requirements
SSP PoC Specification
Current BLE implementation
Validated SSP payload protocol
Available hardware
Planned Android Edge application
Future Cloud/ML roadmap

I recommend using this as the project's master tracking table.

SSP Project Master Roadmap
Step / Milestone	StatusM0. Create project repository and basic structure	✅ DONE
M0. Configure Zephyr development environment	✅ DONE
M0. Configure Nordic nRF52840-DK build/flash/debug workflow	✅ DONE
M0. Verify board flashing and UART debugging	✅ DONE
M0. Configure Android development environment	✅ DONE
M1. Define SSP concept and objectives	✅ DONE
M1. Define Device → Edge architecture	✅ DONE
M1. Define initial SSP monitoring scenario	✅ DONE
M1. Define SSP logical event model	✅ DONE
M1. Define NORMAL and ELEVATED Edge states	✅ DONE
M1. Define first acceptance criteria	✅ DONE
M2. Generate synthetic SSP motion events on Device	✅ DONE
M2. Create Zephyr SSP event subsystem	✅ DONE
M2. Create SSP event publisher	✅ DONE
M2. Implement periodic event generation	✅ DONE
M3. Design SSP BLE service	✅ DONE
M3. Create BLE GATT service	✅ DONE
M3. Create SSP notification characteristic	✅ DONE
M3. Configure BLE advertising	✅ DONE
M3. Implement BLE connection handling	✅ DONE
M3. Implement notification subscription handling	✅ DONE
M3. Design SSP BLE payload format	✅ DONE
M3. Implement payload serialization	✅ DONE
M3. Implement protocol version field	✅ DONE
M3. Implement device ID field	✅ DONE
M3. Implement sequence number field	✅ DONE
M3. Implement timestamp field	✅ DONE
M3. Implement motion field	✅ DONE
M3. Validate payload structure	✅ DONE
M3. Verify Device discovery from Android	✅ DONE
M3. Verify BLE connection establishment	✅ DONE
M3. Verify GATT service discovery	✅ DONE
M3. Verify notification subscription	✅ DONE
M3. Verify BLE notification transport	✅ DONE
M3. Verify packet timing	✅ DONE
M3. Verify sequence integrity	✅ DONE
M3. Perform long-duration notification test	✅ DONE
M3. Create UART monitor and validation tooling	✅ DONE
Deliverable 1 report preparation	✅ DONE (draft level)
Investigate kernel crash at PC=0x000107c4	⏳ TBD
Root-cause analysis using addr2line	⏳ TBD
Verify notification stability under long-duration testing	⏳ TBD
M4. Create Android SSP application skeleton	⏳ TBD
M4. Implement BLE scanning in Android	⏳ TBD
M4. Implement Android BLE connection manager	⏳ TBD
M4. Implement Android GATT discovery	⏳ TBD
M4. Implement Android notification reception	⏳ TBD
M4. Create SSP packet decoder	⏳ TBD
M4. Decode SSP payload on Android	⏳ TBD
M4. Display received SSP events on Android	⏳ TBD
M4. Verify Android receives live SSP packets	⏳ TBD
M5. Receive and integrate IMU hardware	⏳ TBD
M5. Connect IMU to nRF52840-DK	⏳ TBD
M5. Implement IMU driver	⏳ TBD
M5. Read accelerometer measurements	⏳ TBD
M5. Implement motion metric calculation	⏳ TBD
M5. Replace synthetic motion generator with IMU data	⏳ TBD
M5. Generate real MOTION_EVENT from physical movement	⏳ TBD
M5. Validate movement detection	⏳ TBD
M6. Implement SSP state machine on Android	⏳ TBD
M6. Implement NORMAL state	⏳ TBD
M6. Implement ELEVATED state	⏳ TBD
M6. Implement NORMAL → ELEVATED transition	⏳ TBD
M6. Validate complete SSP vertical slice	⏳ TBD
M6. Demonstrate Physical Movement → Edge State Change	⏳ TBD
M7. Add configuration characteristic to BLE service	⏳ TBD
M7. Implement Edge → Device communication	⏳ TBD
M7. Implement adaptive monitoring configuration	⏳ TBD
M7. Demonstrate policy changes based on SSP state	⏳ TBD
M8. Extend SSP logical model with optional position fields	⏳ TBD
M8. Define LOCATION_EVENT	⏳ TBD
M8. Define GEOFENCE_EVENT	⏳ TBD
M9. Integrate GPS sensor	⏳ TBD
M9. Connect GPS to nRF52840-DK	⏳ TBD
M9. Implement NMEA parsing	⏳ TBD
M9. Populate SSP position model	⏳ TBD
M9. Generate location-based events	⏳ TBD
M9. Validate GPS acquisition	⏳ TBD
M10. Implement packet validation layer on Android	⏳ TBD
M10. Validate protocol version	⏳ TBD
M10. Validate sequence continuity	⏳ TBD
M10. Validate timestamp consistency	⏳ TBD
M10. Detect missing packets	⏳ TBD
M11. Implement deterministic risk model	⏳ TBD
M11. Define LOW / MEDIUM / HIGH risk levels	⏳ TBD
M11. Implement rule-based risk scoring	⏳ TBD
M11. Map SSP events to risk levels	⏳ TBD
M12. Build Android Edge intelligence layer	⏳ TBD
M12. Implement feature extraction	⏳ TBD
M12. Implement event aggregation	⏳ TBD
M12. Implement local SSP decisions	⏳ TBD
M13. Define Edge → Cloud API schema	⏳ TBD
M13. Select HTTP or MQTT protocol	⏳ TBD
M13. Document Cloud payload format	⏳ TBD
M14. Implement backend service	⏳ TBD
M14. Implement event ingestion endpoint	⏳ TBD
M14. Implement API validation	⏳ TBD
M15. Implement database	⏳ TBD
M15. Store events	⏳ TBD
M15. Store risk levels	⏳ TBD
M15. Store predictions	⏳ TBD
M16. Implement dashboard	⏳ TBD
M16. Display device status	⏳ TBD
M16. Display event history	⏳ TBD
M16. Display risk evolution	⏳ TBD
M17. Collect training dataset	⏳ TBD
M17. Label SSP events	⏳ TBD
M18. Train first ML model	⏳ TBD
M18. Evaluate accuracy and false positives	⏳ TBD
M18. Integrate ML into Edge processing	⏳ TBD
M19. Implement security validation	⏳ TBD
M19. Validate malformed packets	⏳ TBD
M19. Protect cloud interfaces	⏳ TBD
M20. Perform KPI measurements	⏳ TBD
M20. Measure Device → Edge latency	⏳ TBD
M20. Measure packet loss	⏳ TBD
M20. Measure notification frequency	⏳ TBD
M20. Measure system performance	⏳ TBD
M21. Execute end-to-end SSP demonstrations	⏳ TBD
M21. Execute normal scenario	⏳ TBD
M21. Execute movement scenario	⏳ TBD
M21. Execute elevated-risk scenario	⏳ TBD
M22. Failure testing	⏳ TBD
M22. BLE disconnect recovery	⏳ TBD
M22. Cloud failure recovery	⏳ TBD
M22. Malformed packet handling	⏳ TBD
M23. Final system integration	⏳ TBD
M23. Reproducibility verification	⏳ TBD
Deliverable 2 report	⏳ TBD
Final SSP project demonstration	⏳ TBD
Immediate Next Steps (Highest Priority)
✅ Finalize and submit Deliverable 1 (BLE Communication Report)
⏳ Build Android SSP application (replace nRF Connect)
⏳ Receive and integrate IMU
⏳ Demonstrate Physical Movement → MOTION_EVENT
⏳ Demonstrate MOTION_EVENT → NORMAL→ELEVATED transition
⏳ Complete the first true SSP vertical slice
⏳ Investigate the Zephyr crash as a parallel activity
⏳ Only then proceed to Cloud, Dashboard, GPS, and ML

This table now covers the complete SSP solution from the current state all the way to the final Device → Edge → Cloud → Dashboard → Intelligence architecture.
