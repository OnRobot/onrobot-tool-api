# Supported devices

| Model | Conventional control | Realtime position | Realtime velocity | Realtime force approach | Required realtime firmware |
|---|---|---|---|---|---|
| 2FG7 | ✅ | ✅ | ✅ | ✅ | 1.0.33 |
| 2FG14 | ✅ | ✅ | ✅ | ✅ | 1.1.21 |
| RG2 | ✅ | ✅ | ✅ | ❌ | 1.0.10 |
| RG6 | ✅ | ✅ | ✅ | ❌ | 1.0.10 |
| 3FG15 | ✅\* | ❌ | ❌ | ❌ | — |
| 3FG25 | ✅\* | ❌ | ❌ | ❌ | — |

✅ Supported · ❌ Not available · \* Evaluation only.
Realtime force approach supports both position and velocity approach on 2FG7
and 2FG14.

## Firmware compatibility

Realtime admission checks the connected device against the compatibility
profile included in the SDK, using the versions listed above. The 2FG check
also compares firmware source identity. A matching numeric version alone is
not sufficient; do not bypass an incompatibility error. Newer versions are
not automatically accepted.

The conventional 3FG API is available for evaluation only. It validates product
identity and reads diameter limits, but does not provide realtime control or
firmware identity through the current 3FG API.

## Timing

The parallel-gripper capability profiles permit requested realtime exchange
rates up to 500 Hz. This is not a guarantee of achieved feedback rate or hard
realtime behavior. Transport latency, USB adapter settings and OS scheduling
affect the actual rate. Start with 200 Hz when commissioning realtime control
over a suitable RTU connection and inspect the session's cycle and
missed-deadline counters.

The device exchange period, your command refresh period and any ROS controller
or state-publication rates are separate settings. See
[parallel session configuration](api/parallel-gripper-session.md).
