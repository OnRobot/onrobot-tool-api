# Getting started

Install the library using the [installation guide](installation.md), then
connect with the exact model attached to the Modbus endpoint. Construction
opens the connection and validates the product identity. The 2FG and 3FG APIs
read device limits; the RG API uses installation-specific limits supplied by
the application.

## Read state over Modbus TCP

This first example is read-only and does not command motion:

```cpp
#include <onrobot_tool_api/two_finger_gripper.hpp>

#include <iostream>

int main()
{
    try
    {
        const onrobot::Timeout timeout{ 1000, 1000, 1000 };
        const auto connection = onrobot::tcp(
            "192.0.2.10",
            502,
            onrobot::ModbusSlaveId::Single,
            timeout);

        onrobot::TwoFingerGripper gripper(
            onrobot::Model::TwoFG7,
            connection);

        const auto state = gripper.state();
        std::cout << "External width: " << state.external_width_mm << " mm\n";
    }
    catch (const onrobot::DeviceError &error)
    {
        std::cerr << "Gripper error: " << error.what() << '\n';
        return 1;
    }
}
```

Replace the documentation address with the gripper or Compute Box address.
The default TCP port is 502.

## Modbus RTU

For RTU, replace the connection configuration:

```cpp
const auto connection = onrobot::rtu(
    "/dev/ttyUSB0",
    onrobot::ModbusBaudRate::B115200,
    onrobot::ModbusSlaveId::Single,
    onrobot::Timeout{ 1000, 1000, 1000 });
```

Use `Primary` or `Secondary` instead of `Single` when required by the mounting
configuration. The library configures the supported RTU framing.

## Choose the API

- [2FG7 and 2FG14](api/two-finger-grippers.md): `TwoFingerGripper`
- [RG2 and RG6](api/rg-grippers.md): `RgGripper`
- [3FG15 and 3FG25](api/three-finger-grippers.md): `ThreeFingerGripper`
- [Parallel session](api/parallel-gripper-session.md): managed command and
  recovery lifecycle for 2FG and RG grippers

See [supported devices](supported-devices.md) before using optional or realtime
capabilities.
