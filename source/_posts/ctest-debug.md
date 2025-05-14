---
title: 在 vscode 中是用 CTest 进行单测调试
date: 2025-05-15 01:01:28
categories:
- 技术
tags:
- CMake
---

# 在 vscode 中是用 CTest 进行单测调试

## 问题

如果没有进行特别的配置的话，在 vscode 中的 CTest 是无法进行单测调试的。

![](image1.png)

如上图所示，会显示 `未找到启动配置`。

## 解决方法

在 .vscode 目录中添加一个 launch.json 文件来进行调试配置，文件内容如下

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(ctest) Launch",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${cmake.testProgram}",
            "args": [
                "${cmake.testArgs}"
            ],
            "stopAtEntry": false,
            "cwd": "${cmake.testWorkingDirectory}",
            "console": "integratedTerminal",
        }
    ]
}
```

配置成功后，再进行调试，就成功了。

![](image2.png)