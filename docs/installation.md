# Installation

The provided packages targets Ubuntu 24.04 on amd64. Install the matching
packages for your use case:

Download tool api debian packages from [latest release](https://github.com/OnRobot/onrobot-tool-api/releases/latest)

| Package | Contents |
|---|---|
| `libonrobot-tool-api0` | Shared library required at runtime |
| `libonrobot-tool-api-dev` | Headers, documentation, and CMake integration |

Install both packages to build applications:

```sh
sudo apt install \
  ./libonrobot-tool-api0_<version>_amd64.deb \
  ./libonrobot-tool-api-dev_<version>_amd64.deb
```

The package manager installs required runtime dependencies. An offline system
must already have those dependencies available.

## Use from CMake

Applications require C++17 or newer.

```cmake
cmake_minimum_required(VERSION 3.16)
project(my_gripper_application LANGUAGES CXX)

find_package(onrobot_tool_api REQUIRED CONFIG)

add_executable(my_gripper_application main.cpp)
target_link_libraries(my_gripper_application PRIVATE onrobot::tool_api)
```
