---
title: 跟随鼠标移动的小球
date: 2025-04-06 01:36:56
categories:
- 技术
tags:
- 游戏开发
- C/C++
- EasyX
---

# 跟随鼠标移动的小球

## 绘制边框

EasyX 绘图函数：

```cpp
HWND initgraph(
	int width,          // 窗口宽度
	int height,         // 窗口高度
	int flag = NULL     // 窗口样式
);
```

使用该函数创建窗口后，屏幕会闪过一个窗口后，程序就结束运行了。<br> 通用的方法就是使用一个死循环来卡住程序。

![通用游戏框架](image1.png)

所有的游戏都依赖一个循环来不断更新画面、读入玩家操作事件，而这个循环就叫做游戏的“主循环”

## 绘制小球

`solidcircle` 函数用于画无边框的填充圆

```cpp
void solidcircle(
	int x,          // 圆心 x 坐标
	int y,          // 圆心 y 坐标
	int radius      // 圆的半径
);
```

在窗口的 (300, 300) 位置绘制一个半径为 100 的圆：`solidcircle(300, 300, 100);`

## 处理鼠标移动事件

`peekmessage` 函数用于获取一个消息，并立即返回

```cpp
bool peekmessage(
    ExMessage *msg,         // 指向消息结构体 ExMessage 的指针，用来保存获取到的消息
    BYTE filter = -1,       // 指定要获取的消息范围，默认 -1 获取所有类别的消息
    bool removemsg = true   // 在 peekmessage 处理完消息后，是否将其从消息队列中移除
);


```

在 EasyX 中鼠标的移动、点击或者是键盘的按键操作都称作为“消息”。

![](image2.png)

当我们触发这些消息的时候，EasyX 会将其放置到自己的消息队列中

![](image3.png)

当我们每次调用 peekmessage 函数便尝试从消息队列中拉去一个消息，如果成功拉去到了消息，那么函数则会返回 true，反之则返回 false

![](image4.png)

这样，我们不断地从队列中拉去已有的消息进行处理

![](image5.png)

现在开始就需要对存储消息的结构体进行分析 <[ExMessage 结构体](https://docs.easyx.cn/zh-cn/exmessage)> <br> 查看文档可知，结构体中的 `message` 成员对应的 `WM_MOUSEMOVE` 就是鼠标移动对应的消息。

我们定义两个变量 x 和 y，用来保存圆心位置，当鼠标移动的时候将当前鼠标坐标赋值给它们。这样写好后，就会导致窗口中的圆越来越多。这是因为在绘制新圆的时候，没有将旧的圆擦除。解决的方法很简单，在每次循环绘制圆的之前，将整个窗口清空一次就可以了。

画出圆后屏幕上的圆会不断闪烁，这是因为没有使用双缓冲绘图导致的。在代码中添加三行代码就可以解决：

```cpp
BeginBatchDraw();

while (true) {
    // 游戏主循环
    FlushBatchDraw();
}
EndBatchDraw();
```

[视频教程](https://www.bilibili.com/video/BV1iQ4y1s7Qj?vd_source=d3a8a2f439156e68b612ac4b2fcf649a)