# Changelog

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
