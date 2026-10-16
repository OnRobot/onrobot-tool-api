# Diagnostics and device telemetry

The Tool API exposes read-only device status and telemetry
through the family-specific `diagnostics()` methods. These reads are intended
for monitoring and troubleshooting; they are not a replacement for the
motion-state methods and must not be called from a hard realtime callback.

Every numeric `DiagnosticValue` has a `valid` flag. Check it before using the
value: an invalid value means the reading is unavailable, not that it is zero.
`DiagnosticProvenance` tells you where the value came from. `CommandDerived`
means it came from a command rather than a sensor.

With a session, use `ParallelGripperSession::snapshot()` to read the cached
diagnostics. The session reads diagnostics at the configured interval using
its existing device connection.

## Connection identity

`TwoFingerGripper::identity()` and `RgGripper::identity()` return the identity
read when the device connection was established. For a session you already
own, copy identity and numeric state together without another device read:

```cpp
onrobot::ParallelGripperState state;
onrobot::ParallelGripperIdentity identity;
if (session.trySnapshot(state, identity) && identity.valid) {
  // major/minor/build are numeric firmware version components.
  // Check state.received_at and state.session_state before using motion data.
}
```

This overload tries to acquire its lock once and returns without waiting. If
the lock is busy, both outputs stay unchanged. The session reads identity again
when it reconnects and marks it invalid during recovery or a session fault.
A valid identity describes the last established connection. Check the age of
motion feedback separately.

For 2FG devices, the result also includes board revisions, firmware CRC and five
32-bit source-hash words. Check each field's validity flag. Hash words are in printed
order: encode each as eight hexadecimal digits, retaining leading zeros.
RG supplies product code and firmware version; its unavailable board/CRC/hash
fields are not measurements. The current 3FG API does not provide firmware
identity. None of these fields is a unique device serial number.

## 2FG7 and 2FG14

`TwoFingerGripper::diagnostics()` reports the following information:

| Category | API fields | Units or meaning |
|---|---|---|
| Motion and grip state | status, external/internal width, limits, force, additional results | widths in mm; force in N; decoded status flags |
| Mechanism and device health | raw linear and motor width, 24 V/5 V electrical values, motor speed, counts, temperature | widths in mm; current in A; voltage in V; speed in RPM; temperature in °C |
| Realtime feedback | realtime external width, realtime velocity, realtime force | width in mm; velocity in mm/s; signed force in N |
| Power and force limits | supply power and force ceilings | power in W; force ceilings in N |
| Usage statistics | grip-on time, power cycles, grip cycles, grip-detected count | time in seconds; counters are device counters |

The normal and realtime force fields preserve the firmware sign convention:
closing into an object is negative and opening into an object is positive.
Command force specifies how much closing force to apply. Keep its sign separate
from the feedback convention. Preserve signed feedback; use its absolute value
only when displaying an unsigned magnitude. Do not negate the command.
The status flags report whether the gripper is busy, has detected a grip, needs
calibration, or has a linear sensor error.

## RG2 and RG6

`RgGripper::diagnostics()` reports status bits, electrical values, temperature,
depth and width telemetry, fingertip offset, error code, and signed realtime
linear/angular velocity. `command_force_n` has `CommandDerived` provenance:
it reports the commanded force. RG grippers do not provide measured force
feedback, so do not display this value as contact force. The same status read
includes the safety-switch channels and safety DC-error flag. The 5 V,
safety-24 V, and plug-24 V readings are
reported in volts. `depth_acceleration_mm_s2` remains invalid because the
device does not supply that measurement; it must not be interpreted as zero.

## 3FG15 and 3FG25 (evaluation only)

`ThreeFingerGripper::diagnostics()` reports status flags, diameter and angle
telemetry, electrical values, temperature, diameter limits, fingertip offset,
boost-power limit, and the configured fingertip position. The force field is
`force_percent`, expressed as a percentage; it is not Newtons.

## ROS 2 publication

The ROS 2 `GripperStateBroadcaster` publishes a standard
`diagnostic_msgs/msg/DiagnosticArray` status at `/diagnostics` for a standalone
gripper. Its default topic name is relative, so a namespaced instance publishes
`/<namespace>/diagnostics`; set the broadcaster's `diagnostics_topic` parameter
to remap it. The status contains model, firmware/profile identity, connection
and fault state, sample age, safety flags, health counters, and every available
family-specific diagnostic value as a named key/value entry. Fields that are
not supported by the selected model or explicitly invalidated are omitted.
Cached values can remain present while their age increases; check the status
level and `diagnostic_age` before using them. Use the typed state message to
check each field's validity flag.

For parallel grippers, `GripperState.firmware` and the diagnostic `firmware`
key contain the observed version (plus source hash on 2FG). Invalid or
unsupported identity is an empty typed string and diagnostic `unknown`.
`firmware_source=device-connection` identifies observed information. A supplied
`firmware` configuration label is retained only as `configured_firmware`, never
as the observed identity.

The RG command-force echo is labelled as command-derived in the API and is
never published as measured force. For 3FG devices, the force key is a
percentage. Applications should inspect `status.level`, the diagnostic
message, and the typed state validity fields before acting on a value.
