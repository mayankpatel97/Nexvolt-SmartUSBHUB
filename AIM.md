# Smart USB Hub — Project Feature Specification

**Project status:** Brainstorming  
**Product category:** Programmable USB Hub and Power Management Platform

## 1. Project Overview

The goal is to develop a smart USB hub for embedded development, device testing, and automation. The device will provide independent USB port power control, electrical measurements, automatic recovery, programmable auxiliary power, and PC-based management through a Python SDK.

The hardware will use a transparent enclosure with an integrated OLED display and a clean, minimal appearance.

## 2. USB Hub Functionality

- Four downstream USB ports.
- USB 2.0 High-Speed support (480 Mbps) as the initial target.
- Independent power ON/OFF control for each port.
- USB data connectivity to a host computer.
- Device connection and disconnection detection.
- Host-side USB enumeration monitoring.
- Device inventory, including VID, PID, manufacturer, product name, serial number, and USB speed when available.
- Independent port power control through software.
- Per-port LED status indication.
- Python-controlled LED blinking to identify a physical port.

## 3. USB Port Power Management

- Independent power switching for each downstream port.
- Configurable per-port current limit.
- Default current limit of 500 mA.
- Configurable current limit up to 2 A per port, subject to hardware capability and applicable USB power rules.
- Per-port overcurrent and short-circuit protection.
- Individual fault detection and reporting.
- Remote port power cycling.
- Configurable power-on sequencing and delays.
- Automatic recovery after a detected device failure.
- Configurable recovery retry count, power-off duration, restart delay, and cooldown period.
- Independent data connection control where supported by the selected hub controller.

## 4. Programmable Auxiliary Power Output

One additional two-pin DC output will be provided for powering external electronics and development boards.

### Electrical Requirements

- Output voltage range: 1.8 V to 15 V.
- Maximum output current: 2 A.
- Maximum output power: 30 W at 15 V and 2 A.
- Continuously adjustable output voltage.
- Software-controlled output enable and disable.
- Output voltage measurement.
- Output current measurement.
- Output power calculation.
- Overcurrent protection.
- Short-circuit protection.
- Output overvoltage protection.
- Overtemperature protection.
- Safe startup with the output disabled until initialization.
- Fault reporting and controlled recovery.

### Proposed Implementation

- Dedicated adjustable DC-DC converter, likely a buck-boost topology.
- Voltage adjustment controlled by the management MCU.
- Voltage and current feedback for monitoring and protection.
- Two-pin connector providing VOUT+ and GND.
- Python commands to configure voltage and control the output.

The voltage adjustment resolution, settling time, accuracy, and current-limit behavior will be defined during hardware design.

## 5. Electrical Monitoring and Telemetry

- Measure voltage independently for each USB port.
- Measure current independently for each USB port.
- Calculate power consumption for each port.
- Calculate total downstream USB power consumption.
- Monitor the programmable auxiliary output.
- Monitor external input voltage and available power where practical.
- Detect abnormal current and power consumption.
- Report overcurrent, undervoltage, overvoltage, and thermal faults.
- Provide live measurements through the Python SDK.
- Display electrical measurements on the OLED.
- Record timestamped events and fault information.

## 6. Automatic Recovery and USB Device Testing

- Monitor USB enumeration and device state through host-side software.
- Allow automatic power cycling when a configured failure condition is detected.
- Configure the number of recovery attempts.
- Configure the power-off duration and restart delay.
- Configure cooldown periods to prevent endless power cycling.
- Record recovery attempts and their outcomes.
- Provide automated USB device testing workflows.
- Support sequences such as:
  1. Enable port power.
  2. Wait for device enumeration.
  3. Run a host-side test.
  4. Power-cycle the device if required.
  5. Record the result.

Generic device health checks will depend on host software and the capabilities of the connected device.

## 7. Accelerometer-Based Drop Protection

- Integrate a three-axis accelerometer.
- Detect potential free-fall and impact events.
- Filter normal handling and vibration where possible.
- Switch off all downstream USB outputs when a qualifying drop is detected.
- Record and report the event when communication remains available.
- Latch the shutdown state.
- Require manual or authorized software recovery before restoring power.

The LIS2DW12 is a candidate accelerometer for evaluation, not a finalized component.

## 8. OLED Display and Physical Controls

- Integrate a small OLED display inside the transparent enclosure.
- Display individual USB port states.
- Display port voltage, current, and power.
- Display total power consumption.
- Display auxiliary output voltage and current.
- Show connected-device information where available.
- Show faults, recovery events, and drop-protection status.
- Show firmware version and update progress.
- Automatically cycle through display pages.
- Keep physical controls minimal to preserve a clean appearance.
- Consider one recessed physical button if necessary.

A rotary encoder has not been selected.

## 9. PC Software and Python SDK

Provide a software interface for controlling and monitoring the hub.

### Planned Interfaces

- Python SDK.
- Command-line interface (CLI).
- Local web dashboard.
- USB management interface, with USB CDC ACM as a candidate.
- Shared host-side service for coordinating multiple software clients.

### Planned API Capabilities

- Enable and disable individual USB ports.
- Read port status.
- Read voltage, current, and power measurements.
- Configure per-port current limits.
- Set auxiliary output voltage.
- Enable and disable the auxiliary output.
- Identify a port by blinking its LED.
- Configure automatic recovery policies.
- Retrieve device inventory.
- Retrieve event and fault logs.
- Read and update device configuration.
- Update firmware.

### Illustrative Python API

```python
hub.port_on(1)
hub.port_off(1)

status = hub.get_port_status(1)
measurements = hub.get_measurements(1)

hub.identify_port(1, duration=5)
hub.set_recovery_policy(1, retries=3)

hub.set_voltage(5.0)
hub.output_off()

events = hub.get_events()
hub.update_firmware("hub_firmware.bin")
```

These API names are illustrative and will be finalized during software development.

## 10. Management Firmware

- STM32-class MCU for control, telemetry, safety, and communication.
- Separate USB hub controller for USB data routing.
- Structured command/response protocol.
- Protocol versioning.
- Command identifiers and sequence numbers.
- Payload length and error reporting.
- Optional packet CRC for communication integrity.
- USB-based firmware updates.
- Optional UART bootloader and recovery interface.
- Firmware image validation before installation.
- Protection against interrupted or invalid updates.
- Consider firmware rollback or a protected recovery image.
- Watchdog monitoring.
- Configuration persistence across power cycles.
- Self-test and measurement calibration.
- Firmware version reporting.

The Microchip USB2514B has been discussed as a possible hub controller. Its suitability for runtime control requirements must be verified before selection.

## 11. USB-C Power Input and Power Distribution

- Separate USB-C power-adapter input from the computer's USB data connection.
- Correct USB-C sink detection and input protection.
- Prevent reverse current into the computer's USB VBUS.
- Use external power for downstream USB ports and the auxiliary output.
- Monitor available input power where practical.
- Enforce an overall power budget.
- Evaluate USB-C Power Delivery (PD) negotiation.
- Evaluate a higher-voltage PD input with downstream DC-DC conversion.
- Account for conversion losses, cable ratings, connector ratings, PCB current capacity, and thermal limits.

The power supply must be sized for the combined load if all USB ports and the auxiliary output are used simultaneously.

## 12. Enclosure and Industrial Design

- Use the selected transparent 2.5-inch SATA enclosure as the proposed enclosure.
- Design a custom PCB to fit inside the enclosure.
- Integrate the OLED within the transparent case.
- Keep PCB placement neat and visually appealing.
- Use subtle status LEDs.
- Avoid bulky and unnecessary physical controls.
- Measure internal dimensions, connector openings, mounting features, enclosure ribs, and available height before PCB design.
- Evaluate thermal performance with all power circuits and the OLED installed.

## 13. Potential Future Enhancements

The following are optional ideas for future evaluation:

- Temperature sensing.
- Advanced power-signature and anomaly detection.
- USB traffic monitoring or protocol-analyzer functionality, requiring additional hardware or controller support.
- USB-C Power Delivery monitoring.
- Buzzer or configurable status patterns.
- Network-based remote management.
- Configuration backup and restore.
- Saved power and testing profiles.
- Extended device compatibility and automated test reports.

## 14. Suggested Development Phases

### Phase 1 — Core Hardware

- Four-port USB hub.
- Independent USB port power control.
- Per-port current measurement and protection.
- STM32 management firmware.
- External USB-C power input.
- Programmable 1.8–15 V, 2 A auxiliary output.
- Basic Python control.
- OLED status display.

### Phase 2 — Smart Features

- Automatic power-cycle recovery.
- Device inventory and enumeration monitoring.
- Configurable current limits.
- Power sequencing.
- Timestamped event logs.
- Firmware updates.
- Configuration persistence.
- Accelerometer-based drop protection.

### Phase 3 — Developer Platform

- Python SDK, CLI, and local web dashboard.
- Automated USB device testing.
- Saved test profiles and reports.
- Advanced anomaly detection.
- Optional diagnostic and monitoring features.

## 15. Open Design Decisions

The following decisions remain open:

- Final USB hub controller selection.
- Final MCU selection.
- USB-C PD input voltage and adapter rating.
- Total simultaneous power budget across USB ports and auxiliary output.
- Whether all four USB ports must support 2 A simultaneously.
- Auxiliary output voltage resolution, accuracy, and settling time.
- Current measurement accuracy and sampling rate.
- OLED size, resolution, and interface.
- Accelerometer selection and drop-detection thresholds.
- Enclosure dimensions and final PCB layout.
- USB data-port control capabilities.
- Firmware update and rollback architecture.
- Thermal management and connector selection.

## 16. Design Principle

The product should remain compact, reliable, and easy to use. Hardware protection must operate independently of the PC software wherever practical. The USB management interface should remain available during downstream power shutdowns whenever the power architecture allows it.

All components, interfaces, and optional enhancements remain subject to validation during prototyping.
