# Parallel gripper session API

`ParallelGripperSession` uses one connection to manage communication,
command timing, state updates, Stop and recovery for 2FG and RG. It does not
support 3FG models. Do not mix a session with direct device calls on another
connection to the same gripper.

## Configuration

| Setting | Default | Meaning |
|---|---:|---|
| `model` | `TwoFG7` | Connected 2FG or RG model |
| `connection` | TCP defaults | TCP or RTU configuration |
| `rg_motion_limits` | Not set | Required limits for the fitted RG fingers and workspace |
| `two_finger_supply_power_w` | Not set | Optional 14-48 W budget reapplied on each 2FG connection |
| `conventional_period` | 20 ms | Conventional state-polling interval |
| `diagnostic_refresh_period` | 1000 ms | Optional diagnostic refresh interval in conventional/idle operation |
| `realtime_period` | 2 ms | Requested realtime device exchange interval |
| `realtime_command_timeout` | 100 ms | Required realtime command-refresh timeout |
| `maximum_consecutive_failures` | 3 | Failures before the session faults |

Periods, timeouts and the failure limit must be positive. To request 200 Hz,
set `realtime_period` to `std::chrono::microseconds{5000}`. This does not
configure a ROS controller's update or state-publication rate. Optional
diagnostics are cached during realtime exchanges; inspect their age separately.

<a id="lifecycle-and-admission"></a>

## Starting, stopping and sending commands

| State | Meaning |
|---|---|
| `Configured` | Connected, command processing inactive |
| `Active` | Worker running; new commands can be accepted |
| `Recovering` | Reconnection and identity validation in progress |
| `Faulted` | Motion rejected; the fault remains until recovery succeeds |

| Method | Action |
|---|---|
| `activate()` | Sends Stop, refreshes state and starts the worker |
| `command(...)` | Queues a command; waits for a pending Stop to be applied before accepting new motion |
| `tryCommand(...)` | Returns without waiting; keep the command and retry if the result is `Busy` |
| `stop()` / `tryStop(...)` | Requests Stop; observe its applied sequence before treating it as acknowledged |
| `snapshot()` / `trySnapshot(...)` | Copies the latest cached state without another connection |
| `requestRecovery()` / `tryRequestRecovery(...)` | Requests explicit recovery from a fault |
| `deactivate()` | Requests Stop and waits for the worker to finish |

Accepting a command means it is queued. Check feedback to find out whether the
movement has finished: the applied command sequence, mode, recent measurements,
and busy/grip flags. Stop does not clear a fault. Make the installation safe
before requesting recovery, then send a new motion command after recovery
succeeds. A software acknowledgement alone does not prove the gripper has stopped.

If a fault, recovery or deactivation occurs while a motion call is waiting,
that call cannot start motion later, even after the session is activated again.
Send a new command after recovery succeeds. The waiting call reports `Cancelled`,
or `DeviceFault` while the fault remains. `tryCommand()` does not wait; each
retry is a new request from the caller.

## Observe recovery

`tryRequestRecovery(sequence)` returns `Accepted` only when the faulted session
queues a new recovery. `Busy` leaves the output sequence unchanged. Only one
recovery can be active; repeated `requestRecovery()` calls do not replace it.

Use `trySnapshot(state, identity, recovery)` with
`ParallelGripperRecoveryState` to copy state, identity and recovery status
together, without allocating memory or waiting. If it returns false, all outputs
are unchanged. The optional velocity-calibration overload also accepts this
recovery output.

Match `active_sequence` or `result_sequence` to the accepted request's sequence.
`phase` distinguishes `Queued` from `Running`; a terminal `result` is
`Succeeded`, `Failed`, or `Aborted`, with `result_code` describing the cause.
The last completed result remains available during later commands and recovery
attempts. On success, the worker has checked the connection and feedback and
returned to the active state with no motion command running. On failure, the
fault remains. Shutting down the session aborts an unfinished recovery. Check
the result rather than inferring success from counters or elapsed time.
Resolve the cause of a failed recovery before requesting another one.

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
to idle. Send commands regularly; setting the exchange period alone does not
refresh the command. See the [2FG realtime commands](two-finger-grippers.md#realtime-control)
for force targets, field order, units and live limits.

<a id="process-image"></a>

## Reading the cached state

Check each validity flag and sample age before using a measurement.

| Group | Values |
|---|---|
| Session | Model, state, active mode, firmware compatibility and last error |
| Gripper | Aperture/limits, velocity, signed measured force where available, busy/grip flags and status |
| Mechanism | Linear or angular position/velocity, distinct from task aperture |
| 2FG power | Supply power and separate conventional/realtime force ceilings |
| RG safety | Both safety switches and safety DC error |
| Diagnostics | Cached device readings, validity and source of each value |
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

This example queues a command and then requests Stop; it does not wait for the
grip to finish. Commands run asynchronously. Use recent snapshots to check their
progress. Choose host, force, speed and power appropriate for your installation. Secure the gripper, clear
its workspace and provide an independent means of stopping the equipment.
