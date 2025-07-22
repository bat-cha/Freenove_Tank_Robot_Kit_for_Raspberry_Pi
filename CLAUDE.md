# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Freenove Tank Robot Kit for Raspberry Pi - a Python-based robotics project with client-server architecture for remote control of a tank robot. The project includes motor control, servo control, LED management, camera streaming, ultrasonic sensing, and infrared line following capabilities.

## Hardware Support

The project supports two PCB versions (V1.0 and V2.0) with different GPIO pin mappings:
- **V1.0**: Works best with Raspberry Pi 4 (limited Pi 5 support due to library compatibility)
- **V2.0**: Optimized for Raspberry Pi 5 with hardware SPI and PWM support

Key hardware differences:
- Servo pins: V1.0 uses GPIO7/8, V2.0 uses GPIO12/13
- LED control: V1.0 uses GPIO18 (rpi_ws281x), V2.0 uses GPIO10 (SPI)
- Infrared sensors: V1.0 uses GPIO16/20/21, V2.0 uses GPIO16/26/21

## Development Setup

### Initial Setup
```bash
# Install dependencies and configure hardware
cd Code
python3 setup.py
```

The setup script will:
- Install required Python packages (PyQt5, pigpio, gpiozero, numpy, rpi-hardware-pwm)
- Install hardware libraries (pi-hardware-pwm, rpi-ws281x-python)
- Configure `/boot/firmware/config.txt` for camera and hardware support
- Create hardware parameter file (`params.json`)

### Running the Application

**Server (Raspberry Pi):**
```bash
cd Code/Server
python3 main.py
```

**Client (Control Application):**
```bash
cd Code/Client
python3 Main.py
```

### Testing Individual Components
```bash
# Test ultrasonic sensor
cd Code/Server
python3 car.py Sonic

# Test infrared sensors
python3 car.py Infrared

# Test camera
python3 camera.py

# Test motor control
python3 motor.py

# Test servo control
python3 servo.py

# Test LED control
python3 led.py
```

## Architecture Overview

### Client-Server Model
- **Server**: Runs on Raspberry Pi, manages hardware components and TCP servers
- **Client**: PyQt5 GUI application for remote control via TCP connection

### Core Server Components

**Main Server (`main.py`):**
- Multi-threaded server with separate threads for:
  - Command processing (`threading_cmd_receive`)
  - Video streaming (`threading_video_send`) 
  - Car task management (`threading_car_task`)
  - LED control process (`process_led_running`)

**Hardware Abstraction:**
- `car.py`: High-level car control with autonomous modes (ultrasonic, infrared, clamp operations)
- `motor.py`: Tank motor control with differential steering
- `servo.py`: Camera pan/tilt servo control with hardware version detection
- `camera.py`: Video capture and streaming with configurable resolution
- `led.py`: RGB LED strip control with various patterns
- `ultrasonic.py`: Distance measurement sensor
- `infrared.py`: Line following sensor array

**Communication:**
- `server.py`: TCP server wrapper managing command (port 5003) and video (port 8003) servers
- `tcp_server.py`: Low-level TCP server implementation with client management
- `message.py`: Command parsing and protocol handling
- `command.py`: Command constants and definitions

**Configuration:**
- `parameter.py`: Hardware version detection and parameter management
- `params.json`: Runtime configuration file (auto-generated)

### Operating Modes

1. **Move Mode (Mode 1)**: Manual control with ultrasonic distance reporting
2. **Ultrasonic Mode (Mode 2)**: Autonomous obstacle avoidance
3. **Infrared Mode (Mode 3)**: Line following with object detection and manipulation
4. **Action Modes (4-6)**: Clamp control (stop, up, down)

### Communication Protocol

Commands use format: `CMD_TYPE#param1#param2#...#\n`

Key commands:
- `CMD_MOTOR#left_speed#right_speed#`: Motor control (-2000 to 2000)
- `CMD_SERVO#servo_index#angle#`: Servo control (0/1, 90-150°)
- `CMD_LED#mode#r#g#b#index#`: LED control
- `CMD_MODE#mode_number#`: Operating mode selection
- `CMD_ACTION#action_type#`: Clamp operations

## File Organization

```
Code/
├── setup.py                 # Dependency installation and hardware setup
├── Server/                  # Raspberry Pi server code
│   ├── main.py             # Main server application
│   ├── server.py           # TCP server wrapper
│   ├── car.py              # High-level car control
│   ├── motor.py            # Motor control
│   ├── servo.py            # Servo control
│   ├── camera.py           # Video capture
│   ├── led.py              # LED control
│   ├── ultrasonic.py       # Distance sensor
│   ├── infrared.py         # Line following sensors
│   └── parameter.py        # Hardware configuration
├── Client/                  # PyQt5 control application
│   ├── Main.py             # Main client application
│   ├── Video.py            # Video streaming client
│   ├── Command.py          # Command definitions
│   └── PID.py              # PID controller for tracking
└── Libs/                   # Hardware libraries
    ├── pi-hardware-pwm/    # Pi 5 PWM support
    └── rpi-ws281x-python/  # LED strip library
```

## Development Notes

- Hardware version detection is automatic via `parameter.py`
- Camera configuration requires manual setup during initial installation
- GPIO cleanup is handled automatically on shutdown
- Multi-threading used extensively - be careful with shared state
- LED control runs in separate process for real-time performance
- Video streaming uses binary protocol with frame length headers

## Common Issues

- Pi 5 compatibility: Some libraries require specific versions or alternatives
- Camera setup: Requires reboot after initial configuration
- GPIO permissions: May need to run with appropriate privileges
- Network setup: Ensure wlan0 interface is available for IP detection