---
title: 在VS Code中调试QML文件
date: 2025-06-15 19:03:28
categories:
- 技术
tags:
- Qt/QML
- VS Code
---

# 在VS Code中调试QML文件

## VS Code中写qml文件

1. 安装好Qt插件
2. 配置好CMakeLists文件
```CMake
cmake_minimum_required(VERSION 3.16)

project(MyMusic VERSION 0.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(Qt6 6.5 REQUIRED COMPONENTS Quick)

qt_standard_project_setup(REQUIRES 6.5)

qt_add_executable(MyMusic
    main.cpp
)

qt_add_qml_module(MyMusic
    URI MyMusic
    VERSION 1.0
    QML_FILES
        Main.qml
)

target_link_libraries(MyMusic
    PRIVATE Qt6::Quick
)

```
3. 配置调试文件 luanch.json
```json
{
            "type": "qml",
            "request": "attach",
            "name": "Attach to QML debugger",
            "host": "localhost",
            "port": "12150"
}
```
4. 配置调试启动命令
```shell
-qmljsdebugger=host:127.0.0.1,port:12150,block,services:DebugMessages,QmlDebugger,V8Debugger,QmlInspector
```
5. 调试步骤
![](image.png)

### 注意
如果使用qt插件生成的CMakeLists，发现控制台里面没有输出，输出全部在调试控制台中，可以选在删除下列代码，使调试信息重新输出在集成控制台
```CMake
set_target_properties(your_project PROPERTIES
MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
MACOSX_BUNDLE TRUE
WIN32_EXECUTABLE TRUE
)
```