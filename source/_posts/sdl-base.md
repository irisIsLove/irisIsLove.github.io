---
title: SDL 基础
date: 2025-06-04 21:13:46
categories:
- 技术
tags:
- 音视频
- SDL
---

# SDL 基础

## SDL 视频显示函数

![SDL 视频显示函数介绍](image1.png)

## 直接上代码

```c
#include <SDL.h>

#include <stdio.h>

int
main(int argc, char* argv[])
{
  int run = 1;
  SDL_Window* window = NULL;
  SDL_Renderer* renderer = NULL;
  SDL_Texture* texture = NULL;

  // 显示小方块的大小
  SDL_Rect rect;
  rect.w = 50;
  rect.h = 50;

  // 初始化SDL
  SDL_Init(SDL_INIT_VIDEO);

  // 创建窗口
  window = SDL_CreateWindow("SDL Window",
                            SDL_WINDOWPOS_UNDEFINED,
                            SDL_WINDOWPOS_UNDEFINED,
                            640,
                            480,
                            SDL_WINDOW_OPENGL | SDL_WINDOW_RESIZABLE);
  if (!window) {
    printf("Can't create window, err: %s", SDL_GetError());
    return -1;
  }

  // 创建渲染器
  renderer = SDL_CreateRenderer(window, -1, 0);
  if (!renderer) {
    printf("Can't create renderer, err: %s", SDL_GetError());
    return -1;
  }

  // 创建纹理
  texture = SDL_CreateTexture(
    renderer, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, 640, 480);
  if (!texture) {
    printf("Can't create texture, err: %s", SDL_GetError());
    return -1;
  }

  int showCount = 0;
  while (run) {
    rect.x = rand() % 600;
    rect.y = rand() % 400;

    SDL_SetRenderTarget(renderer, texture);         // 设置渲染目标为纹理
    SDL_SetRenderDrawColor(renderer, 0, 0, 0, 255); // 纹理背景色为黑色
    SDL_RenderClear(renderer);                      // 清空纹理

    SDL_RenderDrawRect(renderer, &rect); // 在纹理上绘制一个矩形
    SDL_SetRenderDrawColor(renderer, 255, 255, 255, 255); // 设置颜色为白色
    SDL_RenderFillRect(renderer, &rect);                  // 在纹理上填充矩形

    SDL_SetRenderTarget(renderer, NULL);           // 设置渲染目标为窗口
    SDL_RenderCopy(renderer, texture, NULL, NULL); // 拷贝纹理到 CPU

    SDL_RenderPresent(renderer); // 输出到目标窗口上
    SDL_Delay(300);
    if (showCount++ > 30) {
      run = 0;
    }
  }

  SDL_DestroyTexture(texture);
  SDL_DestroyRenderer(renderer);
  SDL_DestroyWindow(window);
  SDL_Quit();

  return 0;
}

```