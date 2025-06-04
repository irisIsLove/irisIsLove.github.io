---
title: FFmpeg 基础命令
date: 2025-06-04 13:50:35
categories:
- 技术
tags:
- 音视频
- ffmpeg
---

# FFmpeg 基础命令

## ffmpeg / ffplay / ffprobe 区别

- ffmpeg
  - Hyper fast Audio and Video encoder
  - 超快音视频解码器 (类似爱剪辑)
- ffplay
  - Simple media player
  - 简单媒体播放器
- ffprobe
  - Simple multimedia streams analyzer
  - 简单多媒体流分析器

## 基础使用方法

```shell
# ffmpeg
ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...

# ffplay
ffplay [options] input_file

# ffprobe
ffprobe [OPTIONS] [INPUT_FILE]
```

## 文档查看命令

```shell
# ffmpeg
ffmpeg -h # 基本信息
ffmpeg -h long # 高级信息
ffmpeg -h full # 所有信息

# ffply
ffmpeg -h 

# ffprobe
ffprobe -h
```

## FFmpeg 音视频处理流程

```shell
# 将一个 1920x1080 的 mp4 文件转换成 1280x720 的 flv 文件
ffmpeg -i test_1920x1080.mp4 -acodec copy -vcodec libx264 -s 1280x720 test_1280x720.flv
```
![FFmpeg 音视频处理流程图](image1.png)

## FFmpeg 常用命令分类

![FFmpeg 常用命令分类](image2.png)

- 查看具体分类所支持的参数

```shell
# 语法
ffmpeg -h type=name

# 例子
ffmpeg -h muxer=flv
ffmpeg -h filter=atempo # atempo 调整音频播放速率
ffmpeg -h encoder=libx264
```

## FFplay 播放控制

下列快捷键需要配合 alt 使用
![](image3.png)