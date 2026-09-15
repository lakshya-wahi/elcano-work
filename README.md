# Elcano Drive-by-Wire System

Embedded C++ software for the **Elcano autonomous tricycle's drive-by-wire (DBW) system**, developed as part of autonomous vehicle research at the University of Washington.

The drive-by-wire controller acts as the low-level interface between autonomous or remote driving commands and the vehicle's physical **steering, throttle, and braking systems**. It supports CAN-based communication, manual RC control, hardware-specific configuration, subsystem testing, and onboard telemetry logging.

## My Contributions

My work in this repository focused primarily on:

- Drive-by-wire pin configuration and hardware mapping in `DBW_Pins.h`
- Debugging and fixing steering and throttle behavior
- Implementing onboard SD-card telemetry logging
- Smaller contributions to remote-control and braking functionality

The surrounding repository contains collaborative Elcano drive-by-wire infrastructure developed by the research team.

## System Overview

The drive-by-wire controller sits between higher-level vehicle commands and the tricycle's physical actuators.

```text
      High-Level Autonomous System
                  |
                  | CAN
                  v
        Drive-by-Wire Controller
                  |
       +----------+----------+
       |          |          |
       v          v          v
    Throttle   Steering    Brakes
       |          |          |
       +----------+----------+
                  |
                  v
          Elcano Tricycle
```

The controller can also receive commands from an RC transmitter for manual vehicle operation.

At runtime, the software manages desired speed, braking, and steering values and translates them into commands for the corresponding hardware controllers.

## Core Components

### Vehicle Controller

`DBW/Vehicle.cpp`

The `Vehicle` class coordinates the low-level vehicle systems.

It maintains desired and current values for:

```text
Speed
Brake state
Steering angle
```

and interfaces with:

- Throttle controller
- Steering controller
- Brake controller
- RC controller
- CAN bus
- SD-card telemetry logger
- Real-time clock

The vehicle can operate using commands received over CAN or through the RC-control path.

### Steering

`DBW/SteeringController.cpp`

The steering controller manages the vehicle's steering actuator and wheel-angle feedback.

Vehicle-specific calibration values map wheel-angle sensor readings to physical steering positions.

The configuration includes limits for:

```text
Maximum left turn
Maximum right turn
Straight-ahead sensor position
Steering actuator pulse range
```

Steering behavior can therefore be calibrated to the physical vehicle rather than relying on fixed assumptions about sensor alignment.

### Throttle

`DBW/ThrottleController.cpp`

The throttle controller converts requested vehicle speed into commands for the electric drive system.

The configuration defines:

- Maximum vehicle speed
- Minimum interpreted speed
- Acceleration limits
- PID tuning parameters

PID control is used as part of the vehicle-control implementation.

### Braking

`DBW/Brakes.cpp`

The braking subsystem controls the physical brake hardware.

The system supports separate brake activation and holding behavior, including a configured maximum duration for high-voltage brake activation before transitioning to a holding state.

### Remote Control

`DBW/RC_Controller.cpp`

The RC controller provides a manual-control path for:

- Throttle
- Steering
- Braking

This allows the vehicle to be operated manually in addition to accepting autonomous commands.

## CAN Communication

The drive-by-wire system communicates with other vehicle components through a CAN bus operating at:

```text
500 kbps
```

CAN message identifiers are defined in:

```text
DBW/Can_Protocol.h
```

The protocol includes identifiers for several system components and message types, including:

```text
RC status
High-level status
Low-level status
RC driving commands
High-level driving commands
Vehicle state
LiDAR
Sonar
Camera detections
```

The low-level controller can receive desired vehicle commands and report vehicle state back to the high-level system.

## Hardware Support

The repository contains configuration for multiple generations of the drive-by-wire hardware.

### Version 3

Designed around the:

```text
Arduino Mega
```

with external CAN hardware.

### Version 4

Designed around the:

```text
Arduino Due
```

using the Due's CAN capabilities.

Hardware configuration is controlled through `DBWversion` and the corresponding pin definitions.

## Hardware Pin Management

`DBW/DBW_Pins.h`

This file maps software functionality to the physical drive-by-wire hardware.

Configured signals include:

### Steering

```text
Left wheel-angle sensor
Right wheel-angle sensor
Steering actuator pulse
Steering power
```

### Braking

```text
Brake activation
Brake voltage selection
Brake control
```

### Vehicle State

```text
Wheel rotation
Speedometer
E-bike controller power
```

### Communication

```text
CAN
SPI
```

The mappings differ between Arduino Mega and Arduino Due versions to account for changes in the underlying drive-by-wire hardware.

## Vehicle Configuration

`DBW/Settings.h` / `SettingsTemplate.h`

Vehicle-specific parameters are separated from the main control logic.

Configuration includes:

```text
Maximum vehicle speed
Acceleration limits
Brake timing
Steering limits
Wheel-angle sensor calibration
Wheel diameter
Throttle PID gains
Steering PID gains
```

For example, steering sensor values are calibrated against the physical minimum, maximum, and straight-ahead wheel positions.

This allows the same controller architecture to be adapted to different vehicle hardware and calibration values.

## Telemetry and Data Logging

The drive-by-wire controller includes an SD-card logging system for recording vehicle behavior during testing.

At startup, the system initializes a real-time clock and creates a timestamped CSV file.

Log files use names based on the current date:

```text
MM_DD_XX.CSV
```

where `XX` increments to avoid overwriting previous test runs.

Each telemetry record contains:

```text
epoch_time_s
time_ms
desired_speed_ms
desired_brake
desired_angle
current_speed
current_brake
current_angle
throttle_pulse
steerpulse
brakeHold
steeringVal
steeringAngleRight
```

The logger records both **requested commands and actual controller state**, making it possible to analyze how the physical system responded during vehicle testing.

Data is flushed to the SD card during operation so that recorded telemetry is preserved for debugging and post-run analysis.

## Testing

The repository includes a dedicated `Test/` implementation for testing low-level vehicle subsystems independently.

### Brake Test

Exercises brake activation, holding, and release behavior.

### Throttle Test

Varies the throttle output to test acceleration and deceleration behavior.

### Steering Test

Commands the steering actuator through different directions to test steering response.

These tests allow individual hardware components to be evaluated without requiring the complete autonomous-driving stack.

## Repository Structure

```text
elcano-work/
│
├── DBW/
│   ├── Drive_By_Wire.ino
│   ├── DBW.ino
│   ├── Vehicle.cpp
│   ├── Vehicle.h
│   ├── DBW_Pins.h
│   ├── Settings.h
│   ├── SettingsTemplate.h
│   ├── Can_Protocol.h
│   ├── SteeringController.cpp
│   ├── SteeringController.h
│   ├── ThrottleController.cpp
│   ├── ThrottleController.h
│   ├── Brakes.cpp
│   ├── Brakes.h
│   ├── RC_Controller.cpp
│   └── RC_Controller.h
│
├── Test/
│   ├── Test.ino
│   ├── Test.cpp
│   ├── SteeringController.cpp
│   ├── ThrottleController.cpp
│   ├── Brakes.cpp
│   └── ...
│
├── Documentation/
│   └── README.md
│
└── README.md
```

## Technologies

### Language

- C++

### Embedded Platforms

- Arduino Due
- Arduino Mega

### Vehicle Communication

- CAN
- SPI

### Control

- PID control
- Wheel-angle sensor feedback
- RC control

### Hardware Interfaces

- Steering actuator
- Electric throttle controller
- Brake system
- SD card
- Real-time clock

## Dependencies

The repository documentation identifies the following external Arduino libraries.

### PID

Brett Beauregard's Arduino PID library is used for controller calculations.

### CAN

For Arduino Mega:

```text
CAN_BUS_Shield / Seeed Studio CAN libraries
```

For Arduino Due:

```text
due_can
can_common
```

### Pin Change Interrupt

Used for configurable interrupt behavior on supported Arduino hardware.

### MCP48x2

Provides support for the digital-to-analog converter used by the throttle system.

Additional functionality uses Arduino libraries for components such as SPI, SD-card access, and real-time-clock communication.

## Installation

Clone the repository and install the required Arduino libraries for the target drive-by-wire hardware.

For Arduino Due, ensure that **Arduino SAM Boards (32-bit ARM Cortex-M3)** support is installed through the Arduino IDE Board Manager.

Then select the appropriate board and serial port:

```text
Tools -> Board -> Arduino Due / Arduino Mega
Tools -> Port -> <connected Arduino>
```

The serial monitor is configured for:

```text
115200 baud
```

Hardware-specific settings and calibration values should be verified before running the controller on a vehicle.

## Safety

This repository interfaces with physical steering, throttle, and braking hardware.

It is research software rather than a production automotive control system. Hardware configuration, actuator limits, steering calibration, CAN configuration, and safety systems must be validated for the specific test platform before operation.

## Research Context

This work formed part of the Elcano autonomous-vehicle research platform at the University of Washington.

The project provided experience working across the boundary between software and physical vehicle hardware, including:

- Embedded C++ development
- Hardware/software integration
- CAN communication
- Vehicle-control systems
- Steering and throttle debugging
- Sensor calibration
- Telemetry instrumentation
- Physical-system testing
