---
title: C++ Module 使用
date: 2025-11-19 12:39:08
categories:
- 技术
tags:
- C/C++
---

# C++ Module 使用

## main.cpp 源码

```c++
import std;

auto main() -> int
{
    std::println("Hello, world!");
    std::vector<int> v{1, 2, 3};
    std::println("{}", v);

    auto start = std::chrono::system_clock::now();
    std::println("UCT time: {:%Y-%m-%d %H:%M:%S}", start);
    std::println("Local time: {:%Y-%m-%d %H:%M:%S %Z}",
                 std::chrono::zoned_time{std::chrono::current_zone(), start});
    return 0;
}
```

## g++ 命令行使用

先要进行模块的预编译

```shell
g++ -std=c++23 -fmodules -fmodule-only -c -fsearch-include-path bits/std.cc
```

然后在编译源文件即可

```shell
g++ -std=c++23 -fmodules main.cpp -o main
```

## CMake使用

```CMake
cmake_minimum_required(VERSION 4.1.2)
set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "d0edc3af-4c50-42ea-a356-e2862fe7a444") # 这串玛去github或者gitlab上招对应的cmake版本

project(ModuleTest)

set(CMAKE_CXX_MODULE_STD ON)
set(CMAKE_CXX_STANDARD 23)

add_executable(ModuleTest main.cpp)
target_link_libraries(ModuleTest stdc++exp) # std::println 在mingw中需要这个库
```

## 注意！

貌似vscode中的微软c++插件对module的支持不好，clangd对自家的编译器支持很好，但是gcc就不能用了