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