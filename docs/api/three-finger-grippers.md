# 3FG15 and 3FG25 API — evaluation only

`ThreeFingerGripper` provides conventional diameter and encoder-angle control
for evaluation. It connects, validates product identity and reads diameter
limits. Realtime control is not available. See
[supported devices](../supported-devices.md).

## Diameter coordinates

| Coordinate | Meaning |
|---|---|
| Centre diameter | Diameter between fingertip centres |
| External-grip diameter | Device-reported diameter used to grip around an object |
| Internal-grip diameter | Device-reported diameter used to expand inside an opening |

Use the selected grip type's limits from the connected device. Diameter is not
parallel jaw aperture; fitted fingers and tip offset affect these coordinates.

## Commands

Both diameter and encoder-angle commands support `Grip`, `Move`, `FlexGrip`
and `PowerFlexGrip` modes.

| Parameter | Unit | Valid range |
|---|---|---|
| `grip_type` | — | `External` or `Internal` |
| `diameter_mm` | mm | Device-reported range for the selected grip type |
| `encoder_angle_rad` | rad | 0–65.535; choose a target appropriate for the device |
| `force_percent` | % | 1–100; not Newtons |

```cpp
#include <onrobot_tool_api/three_finger_gripper.hpp>

int main()
{
    onrobot::ThreeFingerGripper gripper(
        onrobot::Model::ThreeFG25, onrobot::tcp("192.168.1.1"));
    gripper.moveDiameter({
        onrobot::ThreeFingerGripType::External,
        onrobot::ThreeFingerDiameterCommand::Move,
        75.0,
        25.0 });
    gripper.stop();
}
```

Replace the host and target with values verified for your installation. Secure
the gripper, clear its workspace and provide an independent means of stopping
the equipment before running motion examples.

## State and diagnostics

`state()` returns busy, grip-detected, force-grip-detected, calibration-valid
and grip-lost flags; centre/compensated/external/internal diameter; force
percentage; angle values; diameter limits; fingertip-position setting and raw
status. Field suffixes identify mm, rad and rad/s.

`diagnostics()` adds validity-tagged voltage, current, temperature, fingertip
offset and boost-power limit. Read temperature and current through diagnostics,
not nonexistent fields on `ThreeFingerState`. The current API does not expose
3FG firmware identity. See [diagnostics](../diagnostics.md).

Invalid arguments, connection, timeout, protocol and device failures throw
`DeviceError`.
