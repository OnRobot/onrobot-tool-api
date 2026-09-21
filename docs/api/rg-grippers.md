# RG2 and RG6 API

`RgGripper` provides conventional and realtime control for RG2 and RG6.
Construction connects and validates product identity. See
[supported devices](../supported-devices.md) for realtime compatibility.

## Coordinates and limits

| Coordinate | Meaning |
|---|---|
| `width_mm` / task aperture | Opening at the fitted fingertips, including fingertip offset |
| `actual_width_mm` | Diagnostic mechanism width without fingertip offset |
| Mechanism angle | Finger mechanism position in radians |

The application supplies safe `RgMotionLimits` for the fitted fingers and
workspace. The minimum must be at least zero and below the configured maximum.

| Absolute model limit | RG2 | RG6 |
|---|---:|---:|
| Maximum width | 110 mm | 160 mm |
| Maximum target force | 40 N | 120 N |

## Conventional control

`RgMoveCommand` contains width in mm and positive target force in N.
Width must stay within the configured limits; force must not exceed the model
maximum. There is no conventional speed-percentage field on RG commands.

```cpp
#include <onrobot_tool_api/rg_gripper.hpp>

int main()
{
    const onrobot::RgMotionLimits safe_limits{ 20.0, 80.0 };
    onrobot::RgGripper gripper(
        onrobot::Model::RG2, onrobot::tcp("192.168.1.1"), safe_limits);
    gripper.move({ 50.0, 20.0 });
    gripper.stop();
}
```

Replace the host, limits and target with values verified for your installation.
Secure the gripper, clear its workspace and provide an independent means of
stopping the equipment before running motion examples. Support the workpiece
before stopping or releasing a grip.

## State and diagnostics

`state()` provides busy/grip flags, both safety-switch channels and safety DC
error, fingertip offset, voltage/current, depth, temperature, compensated and
uncompensated width, mechanism angle/velocity, task velocity, error code and
raw status. Field suffixes identify units: mm, V, A, °C and rad.

`identity()` returns product code and firmware version; `configuration()` reads
long-term power limit. `diagnostics()` provides validity-tagged telemetry.
RG command-force feedback is **command-derived, not measured contact force**;
do not present it as a force-sensor reading. See [diagnostics](../diagnostics.md).

## Realtime control

| Command | Input | Limit |
|---|---|---|
| `RgRealtimePositionCommand` | Fingertip-compensated aperture (mm), positive force (N) | Configured aperture range; model force ceiling; force must encode to a nonzero value |
| `RgRealtimeVelocityCommand` | Signed mechanism angular velocity (rad/s) | RG2: 67.35°/s; RG6: 48.65°/s |

Positive angular velocity opens; negative closes. The velocity command has no
force-target field. Position and conventional control use the same compensated
task-aperture coordinate.

`realtimeCycle()` performs one synchronous command/feedback transaction. It
returns task and mechanism position/velocity, grip and safety flags, and a
force field with explicit validity. It does not create a control loop. Use
[`ParallelGripperSession`](parallel-gripper-session.md) for a managed loop,
command watchdog, Stop and recovery.

## Stop and safety state

Stop does not clear a latched fault. Check safety flags and remove the cause
before explicitly requesting recovery through a session. A safety DC error
requires a device power cycle after making the equipment safe; software cannot
clear it. Software Stop and reported safety state do not replace your
installation's safety system.

Invalid arguments, connection, timeout, protocol and device failures throw
`DeviceError`.
