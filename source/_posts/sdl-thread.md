---
title: SDL 多线程
date: 2025-06-05 13:02:14
categories:
- 技术
tags:
- 音视频
- SDL
- C/C++
---

# SDL 多线程

## SDL 多线程接口

![SDL 多线程接口](image1.png)

## 上代码

```c
#include <SDL.h>

#include <stdio.h>

SDL_mutex* gLock = NULL;
SDL_cond* gCond = NULL;

int
threadWork(void* arg)
{
  SDL_LockMutex(gLock);

  printf("                                 <========== ThreadWork Sleep.\n");
  SDL_Delay(10000);
  printf("                                 <========== ThreadWork Wait.\n");

  SDL_CondWait(
    gCond, gLock); // 进入等待状态，释放锁，等到被唤醒时且锁被释放才会继续进行
  printf(
    "                                 <========== ThreadWork Receive Signal, "
    "Continue to do ~_~!!!.\n");
  printf("                                 <========== ThreadWork Done.\n");

  SDL_UnlockMutex(gLock);
  return 0;
}

int
main(int argc, char* argv[])
{
  gLock = SDL_CreateMutex();
  gCond = SDL_CreateCond();

  SDL_Thread* t = SDL_CreateThread(threadWork, "ThreadWork", NULL);
  if (!t) {
    printf("  %s\n", SDL_GetError());
    return -1;
  }

  for (int i = 0; i < 2; ++i) {
    SDL_Delay(2000);
    printf("MainThread Execute. ==========>\n");
  }
  printf("MainThread LockMutex before. ==========>\n");
  SDL_LockMutex(gLock); // 等待子线程进入 CondWait 状态才能拿到锁
  printf("MainThread ready send signal. ==========>\n");
  printf("MainThread CondSignal before. ==========>\n");
  SDL_CondSignal(gCond); // 获取到锁，发送信号，唤醒等待线程
  printf("MainThread CondSignal after. =========>\n");
  SDL_UnlockMutex(gLock); // 释放锁，子线程才能拿到锁
  printf("MainThread UnlockMutex after. =========>\n");

  // 资源销毁
  SDL_WaitThread(t, NULL);
  SDL_DestroyCond(gCond);
  SDL_DestroyMutex(gLock);

  return 0;
}
```