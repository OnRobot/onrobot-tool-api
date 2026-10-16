# Changelog

## 1.2.1

- Clarify documentation.

## 1.2.0

- Accept firmware at or above each model's supported minimum.

- Cancel waiting motion calls if a fault, recovery or deactivation occurs.
  They cannot start motion after the session is activated again. Add recovery
  request identifiers and keep the last recovery result available, without
  changing existing state layouts.

- Convert 2FG conventional speed percentages to estimated peak aperture speeds
  for a full opening or closing movement. The conversion follows the firmware's
  calculation and uses the requested force, current force limit and device
  calibration. It can select whichever estimated peak is higher, opening or
  closing. A requested speed above the estimated maximum selects 100%; a speed
  below the estimated minimum selects 1%. Requested speeds must be greater than
  zero and must not be NaN or infinity.

## 1.1.0

- Require explicit positive force targets in 2FG realtime position and velocity
  commands. Zero and out-of-range motion targets are rejected before device I/O;
  use the explicit Stop API to stop realtime control.
- Reject RG realtime position force targets that encode to zero or exceed the
  connected model's supported range.
- Improve Stop and recovery handling.
- Improve and expand documentation.

## 1.0.0

- Initial binary SDK and public documentation release.
