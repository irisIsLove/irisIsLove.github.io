---
title: FFmpeg 库入门
date: 2025-06-09 12:49:31
categories:
- 技术
tags:
- 音视频
- ffmpeg
- C/C++
---

# FFmpeg 库入门

## 封装格式相关

- `avformat_alloc_context();` 负责申请一个AVFormatContext结构的内存,并进行简单初始化
- `avformat_free_context();` 释放该结构里的所有东西以及该结构本身
- `avformat_close_input();` 关闭解复用器,关闭后就不再需要使用 avformat_free_context 进行释放
- `avformat_open_input();` 打开输入视频文件
- `avformat_find_stream_info();` 获取音视频文件信息
- `av_read_frame();` 读取音视频包
- `avformat_seek_file();` 定位文件
- `av_seek_frame();` 定位文

![解封装流程图](image1.png)

## 解码器相关

- `avcodec_alloc_context3();` 分配解码器上下文
- `avcodec_find_decoder();` 根据ID查找解码器
- `avcodec_find_decoder_by_name();` 根据解码器名字
- `avcodec_open2();` 打开编解码器
- `avcodec_send_packet();` 发送编码数据包
- `avcodec_receive_frame();` 接收解码后数据
- `avcodec_free_context();` 释放解码器上下文，包含了 avcodec_close()
- `avcodec_close();` 关闭解码器

![解码器流程图](image2.png)

## FFmpeg 数据结构简介

- AVFormatContext
  - 封装格式上下文结构体，也是统领全局的结构体，保存了视频文件封装格式相关信息。
- AVInputFormat、AVOutputFormat
  - 每种封装格式（例如FLV, MKV, MP4, AVI）对应一个该结构体。
- AVStream
  - 视频文件中每个视频（音频）流对应一个该结构体。
- AVCodecContext
  - 编解码器上下文结构体，保存了视频（音频）编解码相关信息。
- AVCodec
  - 每种视频（音频）编解码器(例如H.264解码器)对应一个该结构体。
- AVPacket
  - 存储一帧压缩编码数据。
- AVFrame
  - 存储一帧解码后像素（采样）数据。

## FFmpeg 数据结构之间的关系

![](image3.png)
![](image4.png)
![](image5.png)
![](image6.png)
![](image7.png)

## FFmpeg 数据结构分析

- AVFormatContext
  - iformat：输入媒体的AVInputFormat，比如指向AVInputFormatff_flv_demuxer
  - nb_streams：输入媒体的AVStream 个数
  - streams：输入媒体的AVStream []数组
  - duration：输入媒体的时长（以微秒为单位），计算方式可以参考av_dump_format()函数。
  - bit_rate：输入媒体的码率
- AVInputFormat
  - name：封装格式名称
  - extensions：封装格式的扩展名
  - id：封装格式ID
  - 一些封装格式处理的接口函数,比如read_packet()
- AVStream
  - index：标识该视频/音频流
  - time_base：该流的时基，PTS*time_base=真正的时间（秒）
  - avg_frame_rate：该流的帧率
  - duration：该视频/音频流长度
  - codecpar：编解码器参数属性
- AVCodecParameters
  - codec_type：媒体类型，比如AVMEDIA_TYPE_VIDEO、AVMEDIA_TYPE_AUDIO等
  - codec_id：编解码器类型， 比如AV_CODEC_ID_H264、AV_CODEC_ID_AAC等。
- AVCodecContext
  - codec：编解码器的AVCodec，比如指向AVCodec ff_aac_latm_decoder
  - width, height：图像的宽高（只针对视频）
  - pix_fmt：像素格式（只针对视频）
  - sample_rate：采样率（只针对音频）
  - channels：声道数（只针对音频）
  - sample_fmt：采样格式（只针对音频）
- AVCodec
  - name：编解码器名称
  - type：编解码器类型
  - id：编解码器ID
  - 一些编解码的接口函数，比如int (*decode)()
- AVPacket
  - pts：显示时间戳
  - dts：解码时间戳
  - data：压缩编码数据
  - size：压缩编码数据大小
  - pos:数据的偏移地址
  - stream_index：所属的AVStream
- AVFrame
  - data：解码后的图像像素数据（音频采样数据）
  - linesize：对视频来说是图像中一行像素的大小；对音频来说是整个音频帧的大小
  - width, height：图像的宽高（只针对视频）
  - key_frame：是否为关键帧（只针对视频）
  - pict_type：帧类型（只针对视频） 。例如I， P， B
  - sample_rate：音频采样率（只针对音频）
  - nb_samples：音频每通道采样数（只针对音频）
  - pts：显示时间戳