---
title: SDL YUV 视频播放
date: 2025-06-05 13:38:10
categories:
- 技术
tags:
- 音视频
- SDL
- C/C++
---

# SDL YUV 视频播放

## SDL 视频显示流程

![SDL 视频显示流程](image1.png)

## 上代码

```c
#include <SDL.h>

// 自定义消息类型
#define REFRESH_EVENT (SDL_USEREVENT + 0)
#define QUIT_EVENT (SDL_USEREVENT + 1)

// 定义分辨率
#define YUV_WIDTH 320
#define YUV_HEIGHT 240

int threadExitFlag = 0;

int
refreshVideoTimer(void* data)
{

  while (!threadExitFlag) {
    SDL_Event event;
    event.type = REFRESH_EVENT;
    SDL_PushEvent(&event);
    SDL_Delay(40);
  }

  threadExitFlag = 0;

  SDL_Event event;
  event.type = QUIT_EVENT;
  SDL_PushEvent(&event);

  return 0;
}

int
main(int argc, char* argv[])
{
  if (SDL_Init(SDL_INIT_VIDEO)) {
    printf("Could not initialize SDL - %s\n", SDL_GetError());
    return -1;
  }

  SDL_Event event;                           // 事件
  SDL_Rect rect;                             // 矩形
  SDL_Window* window = NULL;                 // 窗口
  SDL_Renderer* renderer = NULL;             // 渲染
  SDL_Texture* texture = NULL;               // 纹理
  SDL_Thread* timerThread = NULL;            // 刷新线程
  uint32_t pixFormat = SDL_PIXELFORMAT_IYUV; // YUV420P

  // YUV分辨率
  int videoWidth = YUV_WIDTH;
  int videoHeight = YUV_HEIGHT;
  // 窗口分辨率
  int winWidth = YUV_WIDTH;
  int winHeight = YUV_HEIGHT;

  // YUV 文件句柄
  FILE* video = NULL;
  const char* videoPath = "yuv420p_320x240.yuv";

  size_t videoBufferLen = 0;
  uint8_t* videoBuffer = NULL;

  // 帧长度 yuv420p 四个亮度搭一个色度
  uint32_t yFrameLen = videoWidth * videoHeight;
  uint32_t uFrameLen = yFrameLen / 4;
  uint32_t vFrameLen = yFrameLen / 4;
  uint32_t frameLen = yFrameLen + uFrameLen + vFrameLen;

  // 创建窗口
  window = SDL_CreateWindow("SDL YUV Player",
                            SDL_WINDOWPOS_UNDEFINED,
                            SDL_WINDOWPOS_UNDEFINED,
                            winWidth,
                            winHeight,
                            SDL_WINDOW_OPENGL | SDL_WINDOW_RESIZABLE);
  if (!window) {
    printf("SDL: Could not create window - %s\n", SDL_GetError());
    goto END;
  }

  renderer = SDL_CreateRenderer(window, -1, 0);
  texture = SDL_CreateTexture(
    renderer, pixFormat, SDL_TEXTUREACCESS_TARGET, videoWidth, videoHeight);

  videoBuffer = (uint8_t*)malloc(frameLen);
  if (!videoBuffer) {
    printf("Failed to alloc yuv frame space.\n");
    goto END;
  }

  fopen_s(&video, videoPath, "rb");
  if (!video) {
    printf("Failed to open yuv file.\n");
    goto END;
  }

  timerThread = SDL_CreateThread(refreshVideoTimer, "RefreshVideoTimer", NULL);
  if (!timerThread) {
    printf("SDL: Could not create thread - %s\n", SDL_GetError());
    goto END;
  }

  while (1) {
    SDL_WaitEvent(&event);

    if (event.type == REFRESH_EVENT) {
      videoBufferLen = fread(videoBuffer, 1, frameLen, video);
      if (videoBufferLen <= 0) {
        printf("Failed to read data from yuv file.\n");
        goto END;
      }

      // 设置纹理
      SDL_UpdateTexture(texture, NULL, videoBuffer, videoWidth);

      // 显示区域，通过修改宽高来进行缩放
      rect.x = 0;
      rect.y = 0;
      float wRatio = winWidth * 1.0f / videoWidth;
      float hRatio = winHeight * 1.0f / videoHeight;
      rect.w = (int)(videoWidth * wRatio);
      rect.h = (int)(videoHeight * hRatio);

      // 清空渲染器
      SDL_RenderClear(renderer);
      // 渲染纹理
      SDL_RenderCopy(renderer, texture, NULL, &rect);
      // 更新渲染器
      SDL_RenderPresent(renderer);
    } else if (event.type == SDL_WINDOWEVENT) {
      // 获取窗口大小，更新窗口拖动
      SDL_GetWindowSize(window, &winWidth, &winHeight);
      printf("[SDL_WINDOWEVENT] width: %d, height: %d\n", winWidth, winHeight);
    } else if (event.type == SDL_QUIT) {
      threadExitFlag = 1;
    } else if (event.type == QUIT_EVENT) {
      break;
    }
  }

END:
  if (videoBuffer) {
    free(videoBuffer);
  }
  if (video) {
    fclose(video);
  }
  if (timerThread) {
    SDL_WaitThread(timerThread, NULL);
  }
  if (texture) {
    SDL_DestroyTexture(texture);
  }
  if (renderer) {
    SDL_DestroyRenderer(renderer);
  }
  if (window) {
    SDL_DestroyWindow(window);
  }
  SDL_Quit();

  return 0;
}
```