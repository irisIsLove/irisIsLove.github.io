---
title: 线程同步
date: 2025-04-07 15:55:15
categories:
- 技术
tags:
- Linux
- C/C++
---

# 线程同步

## 互斥锁

### 互斥锁使用步骤

1. 创建一把互斥锁 <br> `pthread_mutex_t mutex;`
2. 初始化互斥锁 <br> `	pthread_mutex_init(&mutex);` --- 相当于mutex=1
3. 在代码中寻找共享资源 (也称为临界区) <br> 
```c
pthread_mutex_lock(&mutex);  // mutex = 0
[临界区代码]
pthread_mutex_unlock(&mutex); // mutex = 1
```
4. 释放互斥锁资源 <br> `pthread_mutex_destroy(&mutex);`

注意：必须在所有操作共享资源的线程上都加上锁斗则不能起到同步的效果

### 死锁

死锁并不是 Linux 提供给用户的一种使用方法，而是由于用户是用互斥锁不当引起的一种现象。

#### 常见死锁种类

1. 自己锁自己，如下图代码片段：

![](image3.png)

2. 线程 A 拥有 A 锁，请求获得 B 锁；线程 B 拥有 B 锁，请求获得 A 锁，这样造成线程 A 和线程 B 都不释放自己的锁，而且还想得到对方的锁，从而产生死锁，如下图所示：

![](image4.png)

#### 如何解决死锁

- 让线程按照一定的顺序去访问共享资源
- 在访问其他锁的时候，需要先将自己的锁解开
- 调用 pthread_mutex_trylock，如果加锁不成功会立刻返回

## 读写锁

### 什么是读写锁

读写锁也叫共享 - 独占锁。当读写锁以读模式锁住时，它是以共享模式锁住得到；当它以写模式锁住时，它是以独占模式锁住的。**写独占、读共享。**

### 读写锁使用场合

读写锁非常适合于对数据结构读的次数远大于写的情况。

### 读写锁特性

- 读写锁是 "写模式加锁" 时，解锁前，所有对该锁加锁的线程都会被阻塞。
- 读写锁是 "读模式加锁" 时，如果线程以读模式对其加锁会成功；如果以写模式加锁会阻塞。
- 读写锁时 "读模式加锁" 时，既有试图以写模式加锁的线程，也有试图以读模式加锁的线程。那么读写锁会阻塞随后的读模式请求。优先满足写模式。**读锁、写锁并行阻塞，写锁优先级高。**

总结：读并行，写独占，当读写同时等待锁的时候写的优先级高。

### 读写锁主要操作函数

- 定义一把读写锁：<br> `pthread_rwlock_t rwlock;`
- 初始化读写锁：
  - `int pthread_wrlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);`
  - 函数参数：
    - rwlock - 读写锁
    - attr - 读写锁属性，传 NULL 为默认属性
- 销毁读写锁：<br> `int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);`
- 加读锁：<br> `int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);`
- 尝试加读锁：<br> `int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);`
- 加写锁：<br> `int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);`
- 尝试加写锁：<br> `int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);`
- 解锁：<br> `int pthread_rwlock_unlock(&pthread_rwlock_t *rwlock);`

## 条件变量

条件变量本身不是锁！但它可以造成线程阻塞。通常与互斥锁配合使用。给多线程提供一个会和的场所。

- 使用互斥量保护共享资源
- 使用条件变量可以使线程阻塞，等待某个条件的发生，当条件满足的时候解除阻塞。

### 条件变量的两个动作

- 条件不满足，阻塞线程
- 条件满足，通知阻塞的线程解除阻塞，开始工作

### 条件变量相关函数

- 定义一个条件变量：<br> `pthread_cond_t  cond;`
- 条件变量初始化：
  - 函数原型：`int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);`
  - 函数参数：
    - cond：条件变量
    - attr：条件变量属性，通常传 NULL
  - 函数返回值：成功返回 0，失败返回错误号
- 销毁条件变量：
  - 函数原型：`int pthread_cond_destroy(pthread_cond_t *cond);`
  - 函数参数：条件变量
  - 函数返回值：成功返回 0，失败返回错误号
- 条件变量等待函数：
  - 函数描述：条件不满足，引起线程阻塞并解锁 <br> 条件满足，解除线程阻塞，并加锁
  - 函数原型：`int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);`
  - 函数参数：
    - cond：条件变量
    - mutex：互斥锁变量
  - 函数返回值：成功返回 0，失败返回错误号
- 条件变量唤醒函数：
  - 函数描述：唤醒至少一个阻塞再该条件变量上的线程
  - 函数原型：`int pthread_cond_signal(pthread_cond_t *cond);`
  - 函数参数：条件变量
  - 函数返回值：成功返回 0，失败返回错误号

### 代码片段

![](image5.png)

上述代码中，生产者线程调用 pthread_cond_signal 函数会使消费者线程在 pthread_cond_wait 处解除阻塞。

## 信号量

### 信号量介绍

信号量相当于多把锁，可以理解为是加强版的互斥锁。

### 相关函数

- 定义信号量：`sem_t sem;`
- 初始化信号量：
  - 函数原型：`int sem_init(sem_t *sem, int pshared, unsigned int value);`
  - 函数参数：
    - sem：信号量变量
    - pshared：0 表示线程同步，1 表示进程同步
    - value：最多几个线程操作共享数据
  - 函数返回值：成功返回 0，失败返回 -1，并设置 errno
- 信号量加锁：
  - 函数原型：`int sem_wait(sem_t *sem);`
  - 函数描述：调用该函数一次，相当于 sem--，当 sem 为 0 的时候，引起阻塞
  - 函数参数：信号量变量
  - 函数返回值：成功返回 0，失败返回 -1，并设置 errno
- 信号量解锁：
  - 函数原型：`int sem_post(sem_t *sem);`
  - 函数描述：调用一次，相当于 sem++
  - 函数参数：信号量变量
  - 函数返回值：成功返回 0，失败返回 -1，并设置 errno
- 信号量尝试加锁：
  - 函数原型：`int sem_trywait(sem_t *sem);`
  - 函数描述：尝试加锁, 若失败直接返回, 不阻塞
  - 函数参数：信号量变量
  - 函数返回值：成功返回 0，失败返回 -1，并设置 errno
- 销毁信号量：
  - 函数原型：`int sem_destroy(sem_t *sem);`
  - 函数参数：信号量变量
  - 函数返回值：成功返回 0，失败返回 -1，并设置 errno

### 代码片段

![](image6.png)