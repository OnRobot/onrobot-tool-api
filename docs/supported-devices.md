# Supported devices

The library currently supports these grippers:

| Model | Conventional control | Realtime position | Realtime velocity | Minimum supported firmware |
|---|---|---|---|---|
| 2FG7 | ✅ | ✅ | ✅ | ≥ 1.0.33 |
| 2FG14 | ✅ | ✅ | ✅ | ≥ 1.1.21 |
| RG2 | ✅ | ✅ | ✅ | ≥ 1.0.11 |
| RG6 | ✅ | ✅ | ✅ | ≥ 1.0.11 |
| 3FG15 | ✅ | ❌ | ❌ | ≥ 1.3.21 |
| 3FG25 | ✅ | ❌ | ❌ | ≥ 1.3.21 |

✅ Supported &nbsp;&nbsp; ❌ Not available

Older firmware versions are not supported.

The maximum supported update rate for realtime-enabled models is 500 Hz. The
practical rate may depend on the application and connection.
