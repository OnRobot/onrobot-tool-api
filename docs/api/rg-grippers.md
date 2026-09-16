# RG2 and RG6 API

`RgGripper` provides conventional and realtime control for RG2 and RG6.
Construction connects to the device and validates its product identity.

See [supported devices](../supported-devices.md) for firmware requirements.

## Coordinates

| Coordinate | Meaning |
|---|---|
| Width | Opening at the fitted fingertips, including fingertip offset |
| Task aperture | API name for width in realtime and session commands |
| Actual width | Diagnostic mechanism width without fingertip offset |
| Mechanism angle | Finger mechanism position in radians |

## Motion limits

The application must provide a safe width range when creating the gripper.

| Limit | RG2 | RG6 |
|---|---:|---:|
| Maximum width | 110 mm | 160 mm |
| Maximum force | 40 N | 120 N |

The configured minimum must be zero or greater and below the configured
maximum. Use limits appropriate for the installed fingers and workspace.

## Conventional control

| Parameter | Unit | Valid range |
|---|---|---|
| `width_mm` | mm | Configured motion limits |
| `force_n` | N | Greater than 0 and no more than the model maximum |

```cpp
#include <onrobot_tool_api/rg_gripper.hpp>

int main()
{
    const onrobot::RgMotionLimits safe_limits{ 20.0, 80.0 };
    onrobot::RgGripper gripper(
        onrobot::Model::RG2,
        onrobot::tcp("192.0.2.10"),
        safe_limits);

    gripper.move({ 50.0, 20.0 });
    gripper.stop();
}
```

Always replace the example limits with values verified for the installation.
Secure the gripper, clear its workspace, and provide an independent means of
stopping the equipment before commanding motion.

## State

`state()` returns:

| Field | Meaning |
|---|---|
| `busy` | Motion is active |
| `grip_detected` | An object has been detected |
| `safety_1_pushed` | Safety switch 1 is pushed |
| `safety_1_triggered` | Safety switch 1 is triggered |
| `safety_2_pushed` | Safety switch 2 is pushed |
| `safety_2_triggered` | Safety switch 2 is triggered |
| `safety_dc_error` | Safety circuit reports a DC error |
| `fingertip_offset_mm` | Configured fingertip offset in mm |
| `motor_voltage_v` | Motor voltage in V |
| `motor_current_a` | Motor current in A |
| `actual_depth_mm` | Current depth in mm |
| `actual_relative_depth_mm` | Relative depth in mm |
| `temperature_c` | Temperature in °C |
| `width_mm` | Fingertip-compensated width in mm |
| `actual_width_mm` | Width without fingertip compensation in mm |
| `actual_width_with_fingertip_offset_mm` | Compatibility name for `width_mm` |
| `mechanism_angular_position_rad` | Mechanism angle in rad |
| `mechanism_angular_velocity_rad_s` | Mechanism angular velocity in rad/s |
| `task_velocity_mm_s` | Fingertip opening velocity in mm/s |
| `error_code` | Device error code |
| `raw_status` | Unmodified device status word |

Other available readings:

| Method | Values |
|---|---|
| `identity()` | Product code and firmware version |
| `maximumForceN()` | Model force limit |
| `configuration()` | Long-term power limit |
| `realtimeCycle()` | Position, velocity, force, grip, and safety feedback |

## Realtime control

| Command | Input | Limit |
|---|---|---|
| `RgRealtimePositionCommand` | Fingertip-compensated opening and force | Configured width limits; force between 0 and model maximum |
| `RgRealtimeVelocityCommand` | Signed mechanism angular velocity | RG2: 67.35°/s; RG6: 48.65°/s |

Realtime velocity is supplied in radians per second. Positive values open the
gripper and negative values close it.

`realtimeCycle()` performs one command-and-feedback transaction. Applications
that need a managed loop, command timeout, and recovery can use
[`ParallelGripperSession`](parallel-gripper-session.md).

Invalid values and connection, timeout, protocol, or device failures throw
`DeviceError`.
