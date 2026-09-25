# Changelog

## 1.2.0

- Accept firmware at or above each model's supported minimum.

- Prevent blocked motion calls from crossing fault, recovery or reactivation
  boundaries. Add coherent recovery request identity and retained outcomes
  without changing existing process-image layouts.

- Add force-, live-limit-, device- and direction-aware 2FG conventional
  full-travel peak-speed conversion, with firmware-matched normalization,
  a fastest-direction envelope and positive SI bounds selecting up to 100%.
  Below-minimum positive requests select the slowest native speed (1%).

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
