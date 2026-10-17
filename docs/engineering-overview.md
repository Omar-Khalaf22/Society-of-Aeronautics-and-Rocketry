# Engineering overview

## Purpose and attribution

USF SOAR's NASA Student Launch electronics/software work connects flight and payload hardware to ground operators. Its embedded side acquires sensor data, records evidence, and coordinates control functions; its ground side receives messages and supports monitoring and operator commands.

Omar Khalaf's work centered on leading/contributing to EE/CS integration, telemetry/data acquisition, sensor and communication interfaces, PCB bring-up, and physical testing. The code and hardware designs are team work. This guide maps the archived implementation without assigning every file to one author.

## Read the code in this order

### 1. Sensor acquisition and on-board records

Start with [`drone_master.ino`](../drone_master/drone_master.ino), then the IMU, barometer, SD-card, and LoRa components in the same directory. The main program coordinates those interfaces rather than putting every sensor implementation into the loop.

[`slave_module_rocket.ino`](../slave_module_rocket/slave_module_rocket.ino) provides another view of GPS/IMU acquisition, telemetry scheduling, and SD recording. Local recording and radio transmission are separate paths: on-board storage can retain observations that are not available in the live ground view.

### 2. Deployment and peripheral coordination

[`deploy_module.ino`](../deploy_module/deploy_module.ino) brings together distance and altitude observations, motor control, deployment state, notification, and BLE/LoRa communication. Adjacent classes isolate motor, ranging, barometer, filtering, and radio responsibilities.

The relevant software engineering challenge is coordinating state, timing, sensor observations, and hardware interfaces in one physical system. The archived code is hardware-specific; a state transition in source alone is not evidence that a physical deployment is safe or validated for a new configuration.

### 3. Ground-station decoding and commands

[`SOAR_Echo_Base/app.py`](../SOAR_Echo_Base/app.py) is the application entry point. [`Controllers/control_panel.py`](../SOAR_Echo_Base/Controllers/control_panel.py) handles serial connections, message interpretation, status events, and operator command routing. The snapshot uses Flask/Socket.IO components and background handling for serial data.

The communication boundary matters: raw device messages must be decoded into values/status that the operator interface can display, while commands must be serialized back to the appropriate device. This archive contains components of that workflow, not every dependency needed to run the full interface.

### 4. Connect software to electronics

The [detailed payload architecture](../Media/Payload_design_v5.png) shows how the drone, deployment, recovery, ground-computer, and video paths were designed to connect. The [schematics directory](../Schematics) includes module schematics, PCB Gerbers, and supporting drawings.

The radio telemetry path and the video path are distinct in the design. Do not infer that video is transmitted through the LoRa telemetry channel.

## Useful evidence

| Evidence | What it supports |
| --- | --- |
| Firmware and driver classes | Embedded module boundaries, acquisition/control logic, communication, and logging |
| Ground-station controller | Message handling and operator-command integration |
| Architecture drawing and schematics | System interfaces and electronics design context |
| CSV files | Examples of recorded data formats; they are not by themselves a measured system-reliability benchmark |

The diagrams include broader design intent. Availability of a diagram or source file does not establish that every depicted feature is complete in this snapshot.

## Reproduction notes

- **Firmware:** use the original target board and verify pin mappings, peripheral wiring, power assumptions, sensor calibration, radio configuration, and required libraries. A portable board/library build manifest is not included.
- **File conventions:** some firmware includes use capitalization that differs from filenames. This can affect builds on case-sensitive systems and should be resolved as part of a deliberate hardware/build restoration.
- **Ground station:** the checked-in files import `Services` and `Models` modules and reference interface assets that are not present in this snapshot. The application therefore cannot be reproduced simply by installing Flask and executing `app.py`.
- **Validation:** the project involved physical hardware and integration testing. This presentation refresh reviewed source and existing media; it did not compile firmware, execute hardware commands, or revalidate a flight configuration.

Source is preserved as a historical engineering archive. Making it a reproducible release would require recovering the missing application assets and documenting the original hardware/build environment.
