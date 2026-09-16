<p align="left">
  <img src="docs/images/onrobot-logo.png" alt="OnRobot" width="260">
</p>

# OnRobot Tool API

OnRobot Tool API is a standalone C++17 library for integrating supported
OnRobot grippers over Modbus TCP or Modbus RTU.

## Supported tools

**2FG7 · 2FG14 · RG2 · RG6 · 3FG15 · 3FG25**

See the [supported-device matrix](docs/supported-devices.md) for firmware and
realtime-control support.

## Install

Download tool api debian packages from [latest release](https://github.com/OnRobot/onrobot-tool-api/releases/latest)

The packages target Ubuntu 24.04 on amd64. Install the matching runtime and
development packages:

```bash
sudo apt install \
  ./libonrobot-tool-api0_<version>_amd64.deb \
  ./libonrobot-tool-api-dev_<version>_amd64.deb
```

Then link the installed CMake target:

```cmake
find_package(onrobot_tool_api REQUIRED CONFIG)
target_link_libraries(my_application PRIVATE onrobot::tool_api)
```

## Documentation

| Guide | Contents |
|---|---|
| [Getting started](docs/getting-started.md) | First read-only connection, TCP, and RTU |
| [Installation](docs/installation.md) | Debian packages and CMake integration |
| [Supported devices](docs/supported-devices.md) | Models, firmware, and realtime support |
| [2FG7 and 2FG14](docs/api/two-finger-grippers.md) | Two-finger commands, state, and limits |
| [RG2 and RG6](docs/api/rg-grippers.md) | RG commands, telemetry, and limits |
| [3FG15 and 3FG25](docs/api/three-finger-grippers.md) | Diameter and encoder-angle control |
| [Parallel session](docs/api/parallel-gripper-session.md) | Managed control loop and recovery |

## Safety

Secure the gripper, clear its workspace, and provide an independent means of
stopping the equipment before commanding motion. Software Stop and
communication watchdogs are not safety-rated emergency stops.

## License

Licensed under the [BSD 3-Clause License](LICENSE).
