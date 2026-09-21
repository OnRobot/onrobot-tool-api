# Installation

The packages support Ubuntu 24.04 on amd64 (x86_64). Download the matching
runtime and development packages from the
[latest GitHub Release](https://github.com/OnRobot/onrobot-tool-api/releases/latest).

| Package | Contents |
|---|---|
| `libonrobot-tool-api1` | Shared library required at runtime |
| `libonrobot-tool-api-dev` | Public headers, complete user documentation and CMake integration |

Install both packages to build applications. From the directory containing
the downloaded pair:

```bash
sudo apt install \
  ./libonrobot-tool-api1_<version>_amd64.deb \
  ./libonrobot-tool-api-dev_<version>_amd64.deb
```

Replace `<version>` with the complete Debian version printed in the filenames.
Use the same version and architecture for both packages. The development
package requires the exact matching runtime. The package manager installs
runtime dependencies, including `libmodbus5`; applications do not need Tool
API source or `libmodbus-dev`.

An offline system must already have the declared dependencies available.
The installed documentation starts at
`/usr/share/doc/onrobot_tool_api/README.md` and matches the GitHub guides for
that SDK release.

## Use from CMake

Applications require C++17 or newer. Save `main.cpp` and this `CMakeLists.txt`
in the same directory:

```cmake
cmake_minimum_required(VERSION 3.21)
project(read_gripper LANGUAGES CXX)

find_package(onrobot_tool_api REQUIRED CONFIG)

add_executable(read_gripper main.cpp)
target_link_libraries(read_gripper PRIVATE onrobot::tool_api)
```

Continue with the [read-only example](getting-started.md).

## Header/runtime mismatch

Old workspace libraries selected through `LD_LIBRARY_PATH` can override the
installed runtime. If CMake reports a header/runtime mismatch, remove the
stale library overlay from the shell environment and configure a clean build
directory. Do not combine headers and libraries from different SDK releases.
