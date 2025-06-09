---
title: FFmpeg 内存模型
date: 2025-06-09 14:43:27
categories:
- 技术
tags:
- 音视频
- ffmpeg
- C/C++
---

# FFmpeg 内存模型

- 从现有的Packet拷贝一个新Packet的时候，有两种情况：
  1. 两个Packet的buf引用的是同一数据缓存空间，这时候要注意数据缓存空间的释放问题；
  2. 两个Packet的buf引用不同的数据缓存空间，每个Packet都有数据缓存空间的copy

![数据共享](image1.png) ![数据独立](image2.png)

## 引用计数

- 对于多个AVPacket共享同一个缓存空间，FFmpeg使用的引用计数的机制（reference-count）：
  - 初始化引用计数为0，只有真正分配AVBuffer的时候，引用计数初始化为1
  - 当有新的Packet引用共享的缓存空间时，就将引用计数 +1
  - 当释放了引用共享空间的Packet，就将引用计数-1；引用计数为0时，就释放掉引用的缓存空间AVBuffer
- AVFrame也是采用同样的机制

## 常用 API

![AVPacket 常用 API](image3.png) ![AVFrame 常用 API](image4.png)