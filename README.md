<p align="left">
  <img src="docs/images/onrobot-logo.png" alt="OnRobot" width="260">
</p>

# OnRobot Tool API

Connect a C++17 application to an OnRobot gripper over Modbus TCP or Modbus
RTU. ROS is not required. The SDK provides a shared library, public headers
and CMake integration for Ubuntu 24.04 on amd64 (x86_64).

**2FG7 · 2FG14 · RG2 · RG6**

The 3FG API is available for evaluation only: **3FG15 · 3FG25**.
See [supported devices](docs/supported-devices.md) for capabilities and
realtime firmware compatibility.

## Get started

Download a matching runtime/development pair from the
[latest GitHub Release](https://github.com/OnRobot/onrobot-tool-api/releases/latest).
Follow [installation](docs/installation.md), then
[read your gripper's state](docs/getting-started.md) before commanding motion.

| Guide | Contents |
|---|---|
| [Getting started](docs/getting-started.md) | Build a read-only example; TCP and RTU setup |
| [Installation](docs/installation.md) | Debian packages and CMake integration |
| [Supported devices](docs/supported-devices.md) | Models, firmware compatibility and control modes |
| [2FG7 and 2FG14](docs/api/two-finger-grippers.md) | Widths, commands, signed force and supply power |
| [RG2 and RG6](docs/api/rg-grippers.md) | Fingertip aperture, angular velocity and safety state |
| [3FG15 and 3FG25](docs/api/three-finger-grippers.md) | Evaluation support for diameter and angle commands |
| [Parallel session](docs/api/parallel-gripper-session.md) | Managed loop, watchdog, Stop and recovery |
| [Diagnostics](docs/diagnostics.md) | Measurement validity, provenance and telemetry |
| [Changelog](CHANGELOG.md) | Changes and compatibility notes |

## Offline documentation

The development package installs these same guides under
`/usr/share/doc/onrobot_tool_api/`. Start with `README.md`; relative links and
images use the same layout as the GitHub documentation.

## Safety

Secure the gripper, clear its workspace and provide an independent means of
stopping the equipment before commanding motion. Software Stop and
communication watchdogs are not safety-rated emergency stops. Validate grip
force and payload retention for the fitted fingers and workpiece.

## License

The distributed library, public headers and documentation use the
[BSD 3-Clause License](LICENSE). Third-party components retain their respective
licenses. The SDK implementation source is not part of the distribution.
