# Getting started

Install the SDK using [installation](installation.md), then connect with the
exact model attached to the Modbus endpoint. Construction opens the connection
and validates product identity. Read state before commanding motion.

## Read state over Modbus TCP

Save this as `main.cpp`. This example reads a 2FG7; it does not issue motion.

```cpp
#include <onrobot_tool_api/two_finger_gripper.hpp>
#include <exception>
#include <iostream>

int main(int argc, char **argv) {
  if (argc != 2) {
    std::cerr << "Usage: read_gripper GRIPPER_IP\n";
    return 2;
  }
  try {
    onrobot::TwoFingerGripper gripper(
        onrobot::Model::TwoFG7, onrobot::tcp(argv[1]));
    const auto state = gripper.state();
    const auto diagnostics = gripper.diagnostics();
    std::cout << "Firmware: " << gripper.identity().firmware_revision << '\n';
    std::cout << "External aperture: " << state.external_width_mm << " mm\n";
    if (diagnostics.temperature_c.valid) {
      std::cout << "Temperature: " << diagnostics.temperature_c.value << " C\n";
    }
    if (diagnostics.force_n.valid) {
      std::cout << "Signed measured force: " << diagnostics.force_n.value << " N\n";
    }
  } catch (const std::exception &error) {
    std::cerr << error.what() << '\n';
    return 1;
  }
}
```

Use the `CMakeLists.txt` from [installation](installation.md), then run:

```bash
cmake -S . -B build
cmake --build build --parallel 2
./build/read_gripper 192.168.1.1
```

The Compute Box default IP is `192.168.1.1`; substitute its configured address
if changed and put your computer on the same subnet. Modbus TCP defaults to
port 502; use `onrobot::tcp(argv[1], port)` for another port. A successful run
prints firmware identity, aperture and available telemetry.

## Modbus RTU

Replace the TCP connection expression with a configuration for the actual
serial device:

```cpp
const auto connection = onrobot::rtu(
    "/dev/ttyUSB0",
    onrobot::ModbusBaudRate::B1000000,
    onrobot::ModbusSlaveId::Single,
    onrobot::Timeout{ 1000, 1000, 1000 });
```

Pass `connection` to the gripper constructor. RTU uses 8E1 framing at 115200
or 1000000 bit/s. Choose the baud rate configured on the device; use
`B115200` for 115200 bit/s. The mounting-dependent slave addresses are
`Single` = 65, `Primary` = 66 and `Secondary` = 67. Your user must have
permission to access the serial device.

Only one Modbus master may own the serial bus. Do not run this example while
a ROS hardware process owns the same gripper. Do not call direct gripper
methods concurrently or create a second connection while a managed session
owns the device.

## Choose the API

- [2FG7 and 2FG14](api/two-finger-grippers.md): `TwoFingerGripper`
- [RG2 and RG6](api/rg-grippers.md): `RgGripper`, with installation-specific limits
- [3FG15 and 3FG25 — evaluation only](api/three-finger-grippers.md): `ThreeFingerGripper`
- [Parallel session](api/parallel-gripper-session.md): managed command timing,
  state, Stop and recovery for 2FG and RG

The device classes use mm, mm/s, N and rad as indicated by their field names.
Three-finger commands use diameter, not parallel jaw aperture. Device limits
depend on the fitted fingers. See [supported devices](supported-devices.md)
before using realtime control and [diagnostics](diagnostics.md) before using
optional readings.
