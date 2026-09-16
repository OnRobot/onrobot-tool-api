# 3FG15 and 3FG25 API

`ThreeFingerGripper` provides conventional diameter and encoder-angle control
for 3FG15 and 3FG25. Construction connects to the device, validates its product
identity, and reads its diameter limits.

See [supported devices](../supported-devices.md) for firmware requirements.

## Diameter coordinates

| Coordinate | Meaning |
|---|---|
| Centre diameter | Diameter measured between fingertip centres |
| External diameter | Diameter across the outside of the fingertips |
| Internal diameter | Diameter across the inside of the fingertips |

Use the external diameter for gripping around an object and the internal
diameter for gripping inside an opening. Limits come from the connected device.

## Commands

Both diameter and encoder-angle control provide these modes:

| Mode | C++ value |
|---|---|
| Grip | `Grip` |
| Move | `Move` |
| Flex Grip | `FlexGrip` |
| Power Flex Grip | `PowerFlexGrip` |

Diameter command parameters:

| Parameter | Unit | Valid range |
|---|---|---|
| `grip_type` | — | `External` or `Internal` |
| `diameter_mm` | mm | Device-reported range for the selected grip type |
| `force_percent` | % | Between 1 and 100 |

Encoder-angle command parameters:

| Parameter | Unit | Valid range |
|---|---|---|
| `grip_type` | — | `External` or `Internal` |
| `encoder_angle_rad` | rad | Between 0 and 65.535; use a target appropriate for the device |
| `force_percent` | % | Between 1 and 100 |

```cpp
#include <onrobot_tool_api/three_finger_gripper.hpp>

int main()
{
    onrobot::ThreeFingerGripper gripper(
        onrobot::Model::ThreeFG25,
        onrobot::tcp("192.0.2.10"));

    gripper.moveDiameter({
        onrobot::ThreeFingerGripType::External,
        onrobot::ThreeFingerDiameterCommand::Move,
        75.0,
        25.0 });
    gripper.stop();
}
```

Always secure the gripper, clear its workspace, and provide an independent
means of stopping the equipment before commanding motion.

## State

`state()` returns:

| Field | Meaning |
|---|---|
| `busy` | Motion is active |
| `grip_detected` | A grip has been detected |
| `force_grip_detected` | A force grip has been detected |
| `calibration_valid` | Calibration is valid |
| `grip_lost` | A detected grip has been lost |
| `diameter_mm` | Current centre diameter in mm |
| `diameter_with_tip_offset_mm` | Diameter including fingertip offset in mm |
| `current_external_diameter_mm` | Current external diameter in mm |
| `current_internal_diameter_mm` | Current internal diameter in mm |
| `force_percent` | Current force in percent |
| `encoder_angle_rad` | Encoder-angle value in rad |
| `finger_angle_rad` | Finger angle in rad |
| `approach_angle_rad` | Approach angle in rad |
| `contact_angle_rad` | Contact angle in rad |
| `encoder_angle_velocity_rad_s` | Encoder angular velocity in rad/s |
| `minimum_diameter_mm` | Minimum centre diameter in mm |
| `maximum_diameter_mm` | Maximum centre diameter in mm |
| `minimum_external_diameter_mm` | Minimum external diameter in mm |
| `maximum_external_diameter_mm` | Maximum external diameter in mm |
| `minimum_internal_diameter_mm` | Minimum internal diameter in mm |
| `maximum_internal_diameter_mm` | Maximum internal diameter in mm |
| `fingertip_position` | Device fingertip-position setting |
| `raw_status` | Unmodified device status word |

Temperature and motor current are not exposed by the current 3FG API.

Invalid values and connection, timeout, protocol, or device failures throw
`DeviceError`.
