---
title: SDL 事件处理
date: 2025-06-04 21:44:08
tags:
---

# SDL 事件处理

## SDL 事件处理

![](image1.png)

## 上代码

```c
#include <SDL.h>

#include <stdio.h>

#define CUSTOM_QUIT_EVENT (SDL_USEREVENT + 0) // 用户自定义事件

int
main(int argc, char* argv[])
{
  SDL_Window* window = NULL;
  SDL_Renderer* renderer = NULL;

  // 初始化SDL
  SDL_Init(SDL_INIT_VIDEO);

  // 创建窗口
  window = SDL_CreateWindow("SDL Event Window",
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
  SDL_SetRenderDrawColor(renderer, 255, 0, 0, 255); // 设置窗口为红色
  SDL_RenderClear(renderer);
  SDL_RenderPresent(renderer);

  SDL_Event event;
  int quit = 0;
  for (;;) {
    SDL_WaitEvent(&event); // 等待事件
    switch (event.type) {
      case SDL_KEYDOWN: // 处理键盘按下
        switch (event.key.keysym.sym) {
          case SDLK_a:
            printf("key down a\n");
            break;
          case SDLK_s:
            printf("key down s\n");
            break;
          case SDLK_d:
            printf("key down d\n");
            break;
          case SDLK_q:
            printf("key down q and push quit event\n");
            SDL_Event quitEvent;
            quitEvent.type = CUSTOM_QUIT_EVENT;
            SDL_PushEvent(&quitEvent);
            break;
          default:
            printf("key down 0x%x\n", event.key.keysym.sym);
            break;
        }
        break;
      case SDL_MOUSEBUTTONDOWN: // 处理鼠标按下
        if (event.button.button == SDL_BUTTON_LEFT) {
          printf("mouse down left\n");
        } else if (event.button.button == SDL_BUTTON_RIGHT) {
          printf("mouse down right\n");
        } else {
          printf("mouse down %d\n", event.button.button);
        }
        break;
      case SDL_MOUSEMOTION: // 处理鼠标移动
        printf("mouse move (%d, %d)\n", event.button.x, event.button.y);
        break;
      case CUSTOM_QUIT_EVENT: // 处理自定义事件
        printf("receive quit event\n");
        quit = 1;
        break;
    }

    if (quit) {
      break;
    }
  }

  SDL_DestroyRenderer(renderer);
  SDL_DestroyWindow(window);
  SDL_Quit();

  return 0;
}

```