# 2FG7 and 2FG14 API

`TwoFingerGripper` provides conventional and realtime control for 2FG7 and
2FG14. Construction connects, checks product identity and loads active limits.
See [supported devices](../supported-devices.md) for realtime compatibility.

## Width coordinates

| Coordinate | Meaning |
|---|---|
| External width | Opening used to grip the outside of an object |
| Internal width | Opening used to expand inside a workpiece |
| Task aperture | External width in realtime and parallel-session commands |
| Raw width | Mechanism width used by `gripRaw()` |

Finger configuration affects external/internal limits. Read `limits()` rather
than hardcoding an aperture range. Realtime and parallel-session commands use
external grip; realtime internal grip is not available.

## Conventional control

| Parameter | Unit | Valid range |
|---|---|---|
| `width_mm` | mm | Device-reported limits for the selected grip type |
| `force_n` | N | Zero selects the device minimum; otherwise at least 20 N on 2FG7 or 40 N on 2FG14, up to the live conventional ceiling |
| `speed_percent` | % | 1–100; native command percentage, not a linear mm/s scale |

```cpp
#include <onrobot_tool_api/two_finger_gripper.hpp>

int main()
{
    onrobot::TwoFingerGripper gripper(
        onrobot::Model::TwoFG7, onrobot::tcp("192.168.1.1"));
    gripper.grip(onrobot::GripType::External, 50.0, 40.0, 60.0);
    gripper.stop();
}
```

Replace the host and target with values verified for your installation.
Secure the gripper, clear its workspace and provide an independent means of
stopping the equipment before running motion examples. Support the workpiece
before stopping or releasing a grip.

`move()` accepts the same values in `MoveCommand`. `open()` and `close()` use
the device-reported external-width endpoints, maximum conventional force and
100% speed; use `grip()` or `move()` when selecting force and speed explicitly.

## State and diagnostics

`state()` returns `busy`, `grip_detected`, `not_calibrated`,
`linear_sensor_error`, external/internal width, raw linear mechanism position,
raw motor width, signed force and `raw_status`. Widths are mm; force is N.
`limits()` supplies width limits and conventional/realtime force ceilings.

Closing into an object produces **negative** signed force feedback; opening
into an object produces **positive** feedback. Closing-force targets remain
positive. Preserve the signed feedback; use its absolute value only when
displaying an unsigned magnitude.

`diagnostics()` adds validity-tagged electrical, temperature, status and
statistics data. See [diagnostics](../diagnostics.md).

## Realtime control

| Command | Fields in constructor/aggregate order |
|---|---|
| `RealtimePositionCommand` | Aperture (mm), maximum velocity (mm/s), positive closing force (N) |
| `RealtimeVelocityCommand` | Signed aperture velocity (mm/s), positive closing force (N) |
| `RealtimeForcePositionCommand` | Aperture (mm), closing force (N), maximum approach velocity (mm/s) |
| `RealtimeForceVelocityCommand` | Closing force (N), signed approach velocity (mm/s) |

Position must stay within live external-width limits. Maximum approach velocity
is 0–300 mm/s; signed velocity is −300–300 mm/s. Positive velocity opens and
negative velocity closes. Force targets must be positive: at least 30 N on
2FG7 or 40 N on 2FG14, and no greater than the device's live realtime ceiling
from `limits()`. If no positive ceiling is reported, the SDK uses 95 N for
2FG7 or 196 N for 2FG14. Values that encode to zero are rejected.

One position command with all required arguments:

```cpp
#include <onrobot_tool_api/two_finger_gripper.hpp>

int main()
{
    onrobot::TwoFingerGripper gripper(
        onrobot::Model::TwoFG7, onrobot::tcp("192.168.1.1"));
    const auto feedback = gripper.realtimeCycle(
        onrobot::RealtimePositionCommand{ 50.0, 5.0, 40.0 });
    gripper.stop();
    return feedback.not_calibrated || feedback.linear_sensor_error ? 1 : 0;
}
```

`realtimeCycle()` performs one synchronous command/feedback transaction; it
does not create or refresh a loop. It returns task aperture/velocity, raw
mechanism position, signed measured force and status from the same exchange.
Use the explicit Stop API to leave realtime control, not a zero-force command.
For a managed loop, command watchdog and recovery, use
[`ParallelGripperSession`](parallel-gripper-session.md).

## Supply power

`powerStatus()` reports supply power, `maximum_force_n` for conventional grips
and `maximum_realtime_force_n` for realtime control. The ceilings are distinct;
validate against the one for the selected control mode.

`setSupplyPower()` accepts 14–48 W and verifies the result. The setting returns
to 14 W after a power cycle; apply another budget again when required. A session
can reapply its configured budget after every connection. Select a budget
appropriate for your supply and application; the ROS launch defaults to 48 W.

Invalid arguments, connection, timeout, protocol and device failures throw
`DeviceError`. Grip detection is not calibrated force tracking or a guarantee
of retention; validate the force and workpiece in your application.
