---
title: 在 Windows 环境下的 MinGW 中使用 vcpkg
date: 2025-08-30 11:17:18
categories:
- 技术
tags:
- C/C++
- MinGW
---

# 在 Windows 环境下的 MinGW 中使用 vcpkg

## 前戏

准备好 [vcpkg](https://github.com/microsoft/vcpkg) 和 [MinGW](https://www.mingw-w64.org/downloads/) 就好了
网上这块教程很多，这里就不赘述了

这里是在 vscode 中用 spdlog 为例做演示
## 配置

这里使用的是 vcpkg 的 manifest 模式

- CMake 配置

```json
// settings.json
"cmake.generator": "MinGW Makefiles",
"cmake.configureSettings": {
    "VCPKG_TARGET_TRIPLET": "x64-mingw-dynamic"
}
```

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.15)
project(HelloSpdlog)

set(CMAKE_CXX_STANDARD 17)

find_package(spdlog REQUIRED CONFIG)
add_executable(HelloSpdlog Application.cpp)

target_link_libraries(HelloSpdlog PRIVATE spdlog::spdlog)
```
- vcpk配置

```json
// vcpkg.json
{
    "dependencies": [
        "spdlog"
    ]
}
```

- 程序文件

```cpp
#include <iostream>
#include <spdlog/spdlog.h>

int main()
{
    std::cout << "Hello World!\n";
    spdlog::info("Hello World!");
}
```

- 程序输出

![程序输出](image.png)

## 其他的配置方法

在系统的环境变量设置 MinGW

```shell
VCPKG_DEFAULT_HOST_TRIPLET = x64-mingw-dynamic
VCPKG_DEFAULT_TRIPLET = x64-mingw-dynamic
```

然后使用 vcpkg 安装命令

```shell
vcpkg install spdlog:x64-mingw-dynamic
```