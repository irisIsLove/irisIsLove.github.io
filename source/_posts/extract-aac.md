---
title: AAC ADTS 格式分析
date: 2025-06-13 14:35:23
categories:
- 技术
tags:
- 音视频
---

# AAC ADTS 格式分析

Advanced Audio Coding(⾼级⾳频解码)，是⼀种由MPEG-4标准定义的有损⾳频压缩格式，由Fraunhofer发展，Dolby, Sony和AT&T是主要的贡献者。

## ADTS

全称是Audio Data Transport Stream。是AAC⾳频的传输流格式。AAC⾳频格式在MPEG-2（ISO-13318-7 2003）中有定义。AAC后来⼜被采⽤到MPEG-4标准中。这种格式的特征是它是⼀个有同步字的⽐特流，解码可以在这个流中任何位置开始。它的特征类似于mp3数据流格式。简单说，**ADTS可以在任意帧解码，也就是说它每⼀帧都有头信息**。

![ADTS的一般格式，空白处表示前后帧](image1.png)

有的时候当你编码AAC裸流的时候，会遇到写出来的AAC⽂件并不能在PC和⼿机上播放，很⼤的可能就是AAC⽂件的每⼀帧⾥缺少了ADTS头信息⽂件的包装拼接。
只需要加⼊头⽂件ADTS即可。⼀个AAC原始数据块⻓度是可变的，对原始帧加上ADTS头进⾏ADTS的封装，就形成了ADTS帧。

AAC⾳频⽂件的每⼀帧由ADTS Header和AAC Audio Data组成。结构体如下：
![](image2.png)
注：ADTS Header的长度可能为7字节或9字节，protecttion_absent=0时，9字节，protecttion_absent=1时，为7字节

每⼀帧的ADTS的头⽂件都包含了⾳频的采样率，声道，帧⻓度等信息，这样解码器才能解析读取。
⼀般情况下ADTS的头信息都是7个字节，分为2部分：
- adts_fixed_header()
- adts_variable_header()

其⼀为固定头信息，紧接着是可变头信息。固定头信息中的数据每⼀帧都相同，⽽可变头信息则在帧与帧之间可变。

## adts_fixed_header

![](image3.png)
- syncword: 同步头，总是为0xFFF，all bits must be 1，代表着一个ADTS帧的开始
- ID: MPEG标识符，0表示MPEG-4，1表示MPEG-2
- Layer: always: '00'
- protection_absent: 表示是否误码校验。Warning, set to 1 if there is no CRC and 0 if there is CRC
- profile: 表示使用哪个级别的ACC，如01 Low Complexity(LC) --- AAC LC。有些芯片只支持ACC LC。
  - 在MPEG-2 ACC中定义了三种：
  ![](image4.png)
  profile的值等于Audio Object Type的值减一
  profile = MPEG-4 Audio Object Type - 1
  ![](image5.png)
  ![avcodec中对profile的定义](image6.png)
- sampling_frequency_index: 表示使用的采样率下标，通过这个下标在Sampling Frequencies[]数组中查找得知采样率的值
  ![](image7.png)
- channel_configuration: 表示声道数，比如2表示立体声双声道
  ![](image8.png)

## adts_variable_header

![](image9.png)
- frame_length: 一个ADTS帧的长度包括ADTS头和ACC原始流。
  - frame length, this value must include 7 or 9 bytes of header length: `$acc_frame_length = (protection_absent == 1 ? 7 : 9) + sizeof(AACFrame)`
- adts_buffer_fullness: 0x7FF 说明时码率可变的码流。
- number_of_raw_data_blocks_in_frame: 表示ADTS帧中有`number_of_raw_data_blocks_in_frame + 1`个ACC原始帧。所以说`number_of_raw_data_blocks_in_frame == 0`表示说ADTS帧中有一个ACC数据块

[ADTS 格式分析代码](https://github.com/irisIsLove/LearnFFmpeg/tree/main/FFmpegDemux)