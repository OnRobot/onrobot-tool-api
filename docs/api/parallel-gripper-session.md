# Parallel gripper session API

`ParallelGripperSession` owns one connection and manages communication,
command timing, coherent state, Stop and recovery for 2FG and RG. It does not
support 3FG models. Do not mix a session with direct device calls on another
connection to the same gripper.

## Configuration

| Setting | Default | Meaning |
|---|---:|---|
| `model` | `TwoFG7` | Connected 2FG or RG model |
| `connection` | TCP defaults | TCP or RTU configuration |
| `rg_motion_limits` | — | Required installation-specific limits for RG |
| `two_finger_supply_power_w` | Not set | Optional 14–48 W budget reapplied on each 2FG connection |
| `conventional_period` | 20 ms | Conventional state-polling interval |
| `diagnostic_refresh_period` | 1000 ms | Optional diagnostic refresh interval in conventional/idle operation |
| `realtime_period` | 2 ms | Requested realtime device exchange interval |
| `realtime_command_timeout` | 100 ms | Required realtime command-refresh timeout |
| `maximum_consecutive_failures` | 3 | Failures before the session faults |

Periods, timeouts and the failure limit must be positive. To request 200 Hz,
set `realtime_period` to `std::chrono::microseconds{5000}`. This does not
configure a ROS controller's update or state-publication rate. Optional
diagnostics are cached during realtime exchanges; inspect their age separately.

## Lifecycle and admission

| State | Meaning |
|---|---|
| `Configured` | Connected, command processing inactive |
| `Active` | Worker running; fresh commands may be admitted |
| `Recovering` | Reconnection and identity validation in progress |
| `Faulted` | Failure latched; motion rejected |

| Method | Action |
|---|---|
| `activate()` | Sends Stop, refreshes state and starts the worker |
| `command(...)` | Queues intent; waits for an unapplied Stop before admitting new motion |
| `tryCommand(...)` | Nonblocking admission; retain and retry intent on `Busy` |
| `stop()` / `tryStop(...)` | Requests Stop; observe its applied sequence before treating it as acknowledged |
| `snapshot()` / `trySnapshot(...)` | Copies the latest process image without another connection |
| `requestRecovery()` / `tryRequestRecovery(...)` | Requests explicit recovery from a fault |
| `deactivate()` | Requests Stop and waits for the worker to finish |

Accepted admission is not physical completion. Check applied command sequence,
mode, fresh measurements and busy/grip flags. Stop does not implicitly clear a
latched fault; recovery requires a safe installation and fresh motion intent
after completion. Do not assume physical standstill from software acknowledgement.

A blocking motion call belongs to the session state in which it began. It
cannot become new motion after a fault, recovery or deactivation/reactivation.
Submit fresh intent after recovery succeeds; an expired call reports
`Cancelled` (or `DeviceFault` while the fault remains latched). `tryCommand()`
does not wait; each retry is a new admission attempt owned by the caller.

## Observe recovery

`tryRequestRecovery(sequence)` returns `Accepted` only when the faulted session
queues a new recovery. `Busy` leaves the output sequence unchanged. Only one
recovery can be active; repeated `requestRecovery()` calls do not replace it.

Use `trySnapshot(state, identity, recovery)` with
`ParallelGripperRecoveryState` to copy a coherent observation without allocating
or waiting. If it returns false, all outputs are unchanged. The optional
velocity-calibration overload also accepts this recovery output.

Match `active_sequence` or `result_sequence` to the admitted sequence.
`phase` distinguishes `Queued` from `Running`; a terminal `result` is
`Succeeded`, `Failed`, or `Aborted`, with `result_code` describing the cause.
The last terminal result remains available during later commands and recovery
attempts. Success means the worker validated the connection and feedback and
returned to active idle. Failure keeps the fault latched; shutdown finalization
aborts an unfinished recovery. Counters and elapsed waiting time are not
recovery outcomes. Submit a new request only after resolving a failed attempt.

## Modes and command refresh

The session supports conventional external grip, model-specific realtime
position/velocity, and 2FG force control with position/velocity approach.
Transitioning between active modes sends Stop before starting the new mode.

| Mode | Command refresh timeout applies |
|---|---|
| 2FG realtime position/velocity and force approach | Yes |
| RG realtime position | No; explicitly Stop when leaving the mode |
| RG realtime velocity | Yes |

When an applicable refresh timeout expires, the worker sends Stop and returns
to idle. Refresh intent regularly; a configured exchange period alone does not
refresh the command. See the [2FG realtime command contract](two-finger-grippers.md#realtime-control)
for force targets, field order, units and live limits.

## Process image

Check each validity flag and sample age before using a measurement.

| Group | Values |
|---|---|
| Session | Model, state, active mode, firmware compatibility and last error |
| Gripper | Aperture/limits, velocity, signed measured force where available, busy/grip flags and status |
| Mechanism | Linear or angular position/velocity, distinct from task aperture |
| 2FG power | Supply power and separate conventional/realtime force ceilings |
| RG safety | Both safety switches and safety DC error |
| Diagnostics | Cached family telemetry, validity and provenance |
| Timing | Sample time, sequences, cycles, missed deadlines, watchdog Stops, reconnects and cycle duration |

An invalid measurement is unavailable, not zero. A copied snapshot can be stale;
check `received_at` and session state. RG command force is not measured force.

## Example

```cpp
#include <onrobot_tool_api/parallel_gripper_session.hpp>

int main()
{
    onrobot::ParallelGripperSessionConfig config;
    config.model = onrobot::Model::TwoFG7;
    config.connection = onrobot::tcp("192.168.1.1");
    config.two_finger_supply_power_w = 48;
    config.realtime_period = std::chrono::microseconds{5000};

    onrobot::ParallelGripperSession session(config);
    session.activate();
    session.command(onrobot::ParallelGripCommand{ 50.0, 40.0, 60.0 });
    const auto state = session.snapshot();
    session.stop();
    session.deactivate();
    return state.last_error.empty() ? 0 : 1;
}
```

This illustrates command admission, not waiting for a completed grip. Commands
are asynchronous; use fresh snapshots to observe progress. Choose host, force,
speed and power appropriate for your installation. Secure the gripper, clear
its workspace and provide an independent means of stopping the equipment.
