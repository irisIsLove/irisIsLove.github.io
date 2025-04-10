---
title: 网络编程 (四)
date: 2025-04-10 18:19:17
categories:
- 技术
tags:
- Linux
- C/C++
- 网络编程
---

# 网络编程 (四)

## 多路 IO - poll

- 函数原型：`int poll(struct pollfd *fds, nfds_t nfds, int timeout);`
  - 函数说明：跟 select 相似，监控多路 IO，但 poll 不能跨平台
  - 参数说明：
    - fds：传入传出参数，实际上是一个结构体数组
    - fds.events：
      - POLLIN ---> 读事件
      - POLLOUT ---> 写事件
    - nfds：数组实际有效内容个数
    - timeout：超时时间，单位是毫秒
      - -1：永久阻塞，知道监控的事件发生
      - 0：不管是否有事件发生，立刻返回
      - \>0：直到监控的事件发生或者超时
  - 返回值：
    - 成功：返回就绪事件的个数
    - 失败：返回 -1 <br> 若 timeout = 0，poll 函数不阻塞，且没有事件发生，此时返回 -1，并且 errno = EAGAIN，这种情况不应该视为错误

```c
struct pollfd 
{
   int   fd;         /* file descriptor */   监控的文件描述符
   short events;     /* requested events */  要监控的事件---不会被修改
   short revents;    /* returned events */   返回发生变化的事件 ---由内核返回
};
```
- 说明：
  1. 当 poll 函数返回的时候，结构体当中的 fd 和 event是 没有发生变化，究竟有没有事件发生由 revents 来判断，所以 poll 是请求和返回分离。
  2. struct pollfd 结构体中的 fd 成员若赋值为 -1，则 poll 不会监控
  3. 相对于 select，poll 没有本质上的改变；但是 poll 可以突破 1024 的限制 

## 多路 IO - epoll

将检测文件描述符的变化委托给内核去处理，然后内核将发生变化的文件描述符对应的事件返回给应用程序。

- 函数原型：`int epoll_create(int size);`
  - 函数说明：创建一个树根
  - 参数说明：
    - size：最大节点数，次参数再 linux 2.6.8 已被忽略，但必须传递一个大于 0 的数。
  - 返回值：
    - 成功：返回一个大于 0 的文件描述符，待变整个树的树根
    - 失败：返回 -1，并设置 errno
- 函数原型：`int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`
  - 函数说明：将要监听的节点在 epoll 树上添加，删除和修改
  - 参数说明：
    - epfd：epoll 树根
    - op：
      - EPOLL_CTL_ADD：添加事件节点到树上
      - EPOLL_CTL_DEL：从树上删除事件节点
      - EPOLL_CTL_MOD：修改树上对应的事件节点
    - fd：事件节点对应的文件描述符
    - event：要操作的事件节点
      ```c
      typedef union epoll_data {
        void        *ptr;
        int          fd;
        uint32_t     u32;
        uint64_t     u64;
      } epoll_data_t;

      struct epoll_event {
        uint32_t     events;      /* Epoll events */
        epoll_data_t data;        /* User data variable */
      };
      ```
      event->events 常用的有：
        - EPOLLIN：读事件
        - EPOLLOUT：写事件
        - EPOLLERR：错误事件
        - EPOLLET：边缘触发模式
      event->fd：要监控的事件对应的文件描述符
- 函数原型：`int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);`
  - 函数说明：等待内核返回事件发生
  - 参数说明：
    - epfd：epoll 树根
    - events：传出参数，其实是一个事件结构体数组
    - maxevents：数组大小
    - timeout：
      - -1：表示永久阻塞
      - 0：立即返回
      - \>0：表示超时等待事件
  - 返回值：
    - 成功：返回发生事件的个数
    - 失败：若 timeout = 0，没有事件发生则返回；返回 -1，设置 errno 值

epoll_wait 的 events 是一个传出参数，调用 epoll_ctl 传递给内核什么值，epoll_wait 返回的时候，内核就传回什么值，不糊对 struct event 的结构体变量的值做任何修改

## 进阶 epoll

### epoll 的两种工作模式

LT ：水平触发，高电平代表 1，只要缓冲区有数据，就一直通知
ET ：边缘触发，电平有变化就代表 1，缓冲区有数据就只会通知一次，之后再有数据才会通知。(若是读数据的时候没有读完，则剩余的数据不会再通知，直到有新的数据到来)。

ET 模式由于只通知一次，所以在读的时候要循环读，直到读完，但是当读完之后 read 就会阻塞，所以应该将该文件描述符设置为非阻塞模式

read 函数在非阻塞模式下读的时候，若返回 -1，且 errno 为 EAGAIN，则表示当前资源不可用，也就是说缓冲区无数据 (缓冲区的数据已经读完了)；或者当 read 返回的读到的数据长度小于请求的数据长度时，就可以确定此时缓冲区已没有数据可读了，也就可以认为此时读事件已经处理完毕。

### epoll 反应堆

反应堆：一个小时件出发一系列反应

epoll 反应堆的思想：
- 将描述符，事件，对应的处理方法封装在一起
- 当描述符对应的事件发生了，自动调用处理方法 (原理就是回调函数)

核心思想：在调用 epoll_ctl 函数的时候，将 events 上树的时候，利用 epoll_data_t 的 ptr 成员，将一个文件描述符，事件和回调函数封装成一个结构体，然后让 ptr 指向这个结构体，然后 调用 epoll_wait 函数返回的手，可以的到具体的 events，然后获得 events 结构体中的 events.data.ptr 指针，ptr 中有回调函数，最终可以调用这个回调函数