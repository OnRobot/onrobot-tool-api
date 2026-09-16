# Parallel gripper session API

`ParallelGripperSession` manages communication, command timing, Stop
transitions, state publication, and recovery for 2FG and RG grippers. It does
not support 3FG models.

## Configuration

| Setting | Default | Meaning |
|---|---:|---|
| `model` | `TwoFG7` | Connected 2FG or RG model |
| `connection` | TCP defaults | Modbus TCP or RTU configuration |
| `rg_motion_limits` | — | Installation-specific RG width limits |
| `two_finger_supply_power_w` | Not set | Optional 2FG setting reapplied after connection |
| `conventional_period` | 20 ms | State polling period |
| `realtime_period` | 2 ms | Realtime cycle period |
| `realtime_command_timeout` | 100 ms | Realtime command refresh timeout |
| `maximum_consecutive_failures` | 3 | Failures before the session enters `Faulted` |

Periods, timeouts, and the failure limit must be greater than zero. Optional
2FG supply power must be between 14 and 48 W.

## Lifecycle

| State | Meaning |
|---|---|
| `Configured` | Connected, but command processing is inactive |
| `Active` | Worker is running and accepting commands |
| `Recovering` | Reconnection and identity validation are in progress |
| `Faulted` | Communication failure is latched |

| Method | Action |
|---|---|
| `activate()` | Sends Stop, refreshes state, and starts the worker |
| `command(...)` | Replaces the queued command without performing transport I/O |
| `stop()` | Queues Stop and returns the session to idle mode |
| `snapshot()` | Returns a coherent copy of the latest process image |
| `requestRecovery()` | Requests recovery from a latched fault |
| `deactivate()` | Requests Stop and waits for the worker to finish |

An old motion command is never replayed after recovery.

## Control modes

| Mode | Command |
|---|---|
| Idle | `stop()` |
| Conventional | `ParallelGripCommand` |
| Realtime position | Model-specific realtime position command |
| Realtime velocity | Model-specific realtime velocity command |

Moving between active modes sends a complete Stop before the new mode begins.

## Command refresh

| Model and mode | Refresh timeout applies |
|---|---|
| 2FG realtime position | ✅ |
| 2FG realtime velocity | ✅ |
| RG realtime position | ❌ |
| RG realtime velocity | ✅ |

When the refresh timeout expires, the session sends Stop and returns to idle.

## Process image

Check each validity flag before using its matching measurement.

| Group | Available values |
|---|---|
| Session | Model, state, active mode, last error |
| Gripper | Opening, limits, velocity, force, busy, grip detected, raw status |
| Mechanism | Linear and angular position and velocity |
| 2FG power | Supply power and maximum force |
| RG safety | Both safety switches and safety DC error |
| Timing | Sample time, sequence numbers, cycle counts, missed deadlines, watchdog Stops, reconnects, cycle duration |

## Example

```cpp
#include <onrobot_tool_api/parallel_gripper_session.hpp>

int main()
{
    onrobot::ParallelGripperSessionConfig config;
    config.model = onrobot::Model::TwoFG7;
    config.connection = onrobot::tcp("192.0.2.10");

    onrobot::ParallelGripperSession session(config);
    session.activate();
    session.command(onrobot::ParallelGripCommand{ 50.0, 40.0, 60.0 });

    const auto state = session.snapshot();

    session.stop();
    session.deactivate();
    return state.last_error.empty() ? 0 : 1;
}
```

Commands are processed asynchronously. Use `snapshot()` to observe application
state instead of assuming a command has completed when `command()` returns.

Always secure the gripper, clear its workspace, and provide an independent
means of stopping the equipment before commanding motion.
