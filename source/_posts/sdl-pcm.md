---
title: SDL 播放 PCM 音频
date: 2025-06-05 16:22:02
categories:
- 技术
tags:
- 音视频
- SDL
- C/C++
---

# SDL 播放 PCM 音频

## 打开音频设备

![](image1.png)

## SDL_AudioCallback

![](image2.png)

## 上代码

```c
#include <SDL.h>

// 每次读取 2 帧数据，以 1024 个采样点为单位 2 通道 16bit 采样点
#define PCM_BUFFER_SIZE (1024 * 2 * 2 * 2)

static uint8_t* sAudioBuf = NULL; // 音频 PCM 数据缓存
static uint8_t* sAudioPos = NULL; // 目前读取的位置
static uint8_t* sAudioEnd = NULL; // 缓存结束的位置

void // 音频设备回调函数
onFillAudioPcm(void* userdata, uint8_t* stream, int len)
{
  SDL_memset(stream, 0, len);

  // 数据读取完毕
  if (sAudioPos >= sAudioEnd) {
    return;
  }

  // 数据够了就读预设长度，不够就只读部分
  int remainBufLen = (int)(sAudioEnd - sAudioPos);
  len = (len < remainBufLen) ? len : remainBufLen;

  // 拷贝数据到 stream 并调整音量
  SDL_MixAudio(stream, sAudioPos, len, SDL_MIX_MAXVOLUME / 8);
  printf("[onFillAudioPcm] len: %d\n", len);
  sAudioPos += len;
}

int
main(int argc, char* argv[])
{
  FILE* audio = NULL;
  const char* audioPath = "44100_16bit_2ch.pcm";

  if (SDL_Init(SDL_INIT_AUDIO)) {
    printf("Could not initialize SDL - %s\n", SDL_GetError());
    return -1;
  }

  fopen_s(&audio, audioPath, "rb");
  if (!audio) {
    printf("Failed to open pcm file.\n");
    goto END;
  }

  sAudioBuf = (uint8_t*)malloc(PCM_BUFFER_SIZE);

  // 打开音频设备
  SDL_AudioSpec audioSpec;
  audioSpec.freq = 44100;          // 采样频率
  audioSpec.format = AUDIO_S16SYS; // 采样点格式
  audioSpec.channels = 2;          // 通道数
  audioSpec.samples = 1024; // 每次读取的样本数，多久产生一次回调和 samples 有关
                            // 用 freq / sample 算出最多延迟多少毫秒
  audioSpec.callback = onFillAudioPcm;
  audioSpec.userdata = NULL;
  if (SDL_OpenAudio(&audioSpec, NULL)) {
    printf("Failed to open audio device - %s\n", SDL_GetError());
    goto END;
  }

  // 播放音频
  SDL_PauseAudio(0);

  size_t dataCount = 0;
  while (1) {
    // 从文件中读取数据
    size_t readBufLen = fread(sAudioBuf, 1, PCM_BUFFER_SIZE, audio);
    if (readBufLen <= 0) {
      break;
    }

    dataCount += readBufLen; // 统计读取的数据总字节数
    printf("Now palying %10zu bytes.\n", dataCount);
    sAudioEnd = sAudioBuf + readBufLen; // 更新 buffer 起始位置
    sAudioPos = sAudioBuf;

    while (sAudioPos < sAudioEnd) {
      SDL_Delay(2); // 等待 PCM 数据消耗
    }
  }
  printf("Play over.\n");
  SDL_CloseAudio();

END:
  if (audio) {
    fclose(audio);
  }
  if (sAudioBuf) {
    free(sAudioBuf);
  }
  SDL_Quit();

  return 0;
}
```