# 2FG7 and 2FG14 API

`TwoFingerGripper` provides conventional and realtime control for 2FG7 and
2FG14. Construction connects to the device, checks its product identity, and
loads its active limits.

See [supported devices](../supported-devices.md) for firmware requirements.

## Width coordinates

| Coordinate | Meaning |
|---|---|
| External width | Width used when gripping the outside of an object |
| Internal width | Width used when expanding inside an opening |
| Task aperture | API name for external width in realtime and session commands |
| Raw width | Unadjusted mechanism width used by `gripRaw()` |

External and internal limits depend on the installed fingers. Read them with
`limits()` instead of hardcoding them.

## Conventional control

| Parameter | Unit | Valid range |
|---|---|---|
| `width_mm` | mm | Device-reported limit for the selected grip type |
| `force_n` | N | Between 0 and the device-reported maximum |
| `speed_percent` | % | Between 10 and 100 |

```cpp
#include <onrobot_tool_api/two_finger_gripper.hpp>

int main()
{
    onrobot::TwoFingerGripper gripper(
        onrobot::Model::TwoFG7,
        onrobot::tcp("192.0.2.10"));

    gripper.grip(onrobot::GripType::External, 50.0, 40.0, 60.0);
    gripper.stop();
}
```

`move()` accepts the same values in a `MoveCommand`. `open()` and `close()` use
the device-reported external-width limits.

## State

`state()` returns:

| Field | Meaning |
|---|---|
| `busy` | Motion is active |
| `grip_detected` | An object has been detected |
| `not_calibrated` | The gripper reports that calibration is required |
| `linear_sensor_error` | The linear sensor reports an error |
| `external_width_mm` | Current external width in mm |
| `internal_width_mm` | Current internal width in mm |
| `linear_mechanism_position_mm` | Raw linear mechanism position in mm |
| `raw_motor_width_mm` | Diagnostic motor width in mm |
| `force_n` | Current force in N |
| `raw_status` | Unmodified device status word |

Other available readings:

| Method | Values |
|---|---|
| `limits()` | External and internal width limits, supply power, and maximum force |
| `powerStatus()` | Current supply power and resulting maximum force |
| `realtimeCycle()` | Opening, velocity, raw mechanism position, and force |

Always secure the gripper, clear its workspace, and provide an independent
means of stopping the equipment before commanding motion.

## Realtime control

`realtimeCycle()` performs one command-and-feedback transaction. Supported
commands are:

| Command | Input |
|---|---|
| `RealtimePositionCommand` | Target external opening and maximum velocity |
| `RealtimeVelocityCommand` | Signed opening velocity |

Position must stay within the reported external-width limits. Realtime
velocity must be between -300 and 300 mm/s. Each call returns the opening,
velocity, raw mechanism position, and force measured in the same cycle.

The method does not create a control loop. Applications that need a managed
loop, command timeout, and recovery can use
[`ParallelGripperSession`](parallel-gripper-session.md).

## Supply power

`powerStatus()` reads the current supply-power setting and force ceiling.
`setSupplyPower()` accepts values between 14 and 48 W and verifies the result.
The setting returns to 14 W after a power cycle, so applications requiring
another value must apply it again.

Invalid values and connection, timeout, protocol, or device failures throw
`DeviceError`.
