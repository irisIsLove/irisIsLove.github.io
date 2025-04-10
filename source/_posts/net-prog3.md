---
title: 网络编程 (三)
date: 2025-04-10 10:55:07
categories:
- 技术
tags:
- Linux
- C/C++
- 网络编程
---

# 网络编程 (三)

## TCP 状态转换

![TCP 状态转换图](image1.png)

说明：上图中粗线表示主动方，虚线便是被动方，细线部分表示一些特殊情况，了解即可，不必深入研究
对于建立连接的过程客户端属于主动方，服务端数据被动接收方 (图的上半部分)
对于关闭 (图的下半部分)，客户端和服务端都可以先进行关闭
处于 ESTABLISHED 状态的升级后就可以收发数据了，双方在通信过程中一直处于 ESTABLISHED 状态，数据传输期间没有状态变化。

TIME_WAIT 状态一定是出现在主动关闭的一方
主动关闭的 socket 端会进入 TIME_WAIT 状态，并且持续 2MSL 时间长度，MSL 就是 maximum segment lifetime (最大分节生命期)，这是一个 IP 数据包能在互联网上生存的最长时间，超过这个时间将在网络中消失。

使用 `netstat -anp` 可以查看连接状态

![](image2.png)
注：数据传输的时候带了一个字节的数据，所以 server 发送给client 的 ACK = x + 2

为什么需要 2MSL？

1. 让四次挥手的过程更可靠，确保最后一个发送给对方的 ACK 到达；若对方没有收到 ACK 应答，对方再次发送 FIN 请求关闭，此时在 2MSL 时间内被动关闭方仍然可以发送 ACK 给对方
2. 为了保证在 2MSL 时间内，不能启动相同的 socket-pair。TIME_WAIT 一定是出现在主动关闭的乙方，也就是说 2MSL 是针对主动关闭的一方来说的；由于 TCP 有可能存在丢包重传，丢包重传若发给了已经断开连接之后相同的 socket-pair (该连接是新建的，与原来的 socket-pair 完全相同，双方使用的是相同的 IP 和端口)，这样会对之后的连接造成困扰，严重可能引起程序异常。

## 端口复用

当出现 `bind error: Address already in use` 这样的问题后，就可以使用端口复用来解决

函数原型：`int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);`
使用例子：`setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(int));`

具体函数说明可以参考 《UNIX 环境高级编程》

## 半关闭状态

### 半关闭的概念

如果一方 close，另一方没有 close，则认为是半关闭状态，处于半关闭状态的时候，可以接收数据，但是不能发送数据。相当于把文件描述符的写缓冲区操纵关闭了。
注意：半关闭一定是出现在主动关闭的一方

### shutdown 函数

长连接和短链接的概念：<br> 连接建立后一直不关闭为长连接 <br> 连接收发数据完毕之后就关闭为短链接

### shutdown 和 close 的区别

shutdown 能够把文件描述符上的读或者写操作关闭，而 close 关闭文件描述符只是将连接的引用计数的值减一，当减到零就真正关闭文件描述符了。

如：调用 dup 函数或者 dup2 函数可以复制一个文件描述符，close 其中一个并不影响另一个文件描述符，而 shutdown 就不同了，一旦 shutdown 了其中一个文件描述符，对所有的文件描述符都有影响。

## 心跳包

一般心跳包用于长连接，用于检查与对方网络连接是否正常

- 方法一：
  ```c
    int keepAlive = 1;
    setsockopt(listenfd, SOL_SOCKET, SO_KEEPALIVE, (void*)&keepAlive, sizeof(keepAlive));
  ```
  由于不能实时的检查网络情况，一般不用这种方法
- 方法二： <br> 在应用程序中自己定义心跳包，使用灵活，能够实时把控。

## 高并发服务器模型 -- select

多路 IO 技术：select，同时监听多个文件描述符，将监控的操作交给内核去处理。

数据类型：
fd_set：文件描述符集合 -- 本质是位图

位图操作函数：
- `void FD_CLR(int fd, fd_set *set);` <br> 将 fd 从  set 集合钟清楚
- `int FD_ISSET(int fd, fd_set *set);` <br> 判断 fd 是否在集合中 <br> 如果 fd 在 set 集合中，返回 1，否则返回 0.
- `void FD_SET(int fd, fd_set* set);` <br> 将 fd 设置到 set 集合中
- `void FD_ZERO(fd_set *set);` <br> 初始化set集合
 
`int select(int nfds, fd_set * readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);`
- 函数介绍：委托内核监听该文件描述符对应的读、写或者错误事件的发生
- 参数说明：
  - nfds：最大的文件描述符 + 1
  - readfds：读集合，是一个传入传出参数 <br> 传入：指的是告诉内核哪些文件描述符需要监控 <br> 传出：指的是内核告诉应用程序哪些文件描述符发生了变化。
  - writefds：写入文件描述符集合 (传入传出参数)
  - exceptfds：异常文件描述符集合 (传入传出参数)
  - timeout：
    - NULL -- 表示永久阻塞，直到有事件发生
    - 0 -- 表示不阻塞，立刻返回，不管是否有监听的事件发生
    - \>0 -- 到指定事件或者有事件发生了就返回
- 返回值：<br> 成功返回发生变化的文件描述符的个数 <br> 失败返回 -1，并设置 errno

调用 select 函数其实就是委托内核帮我们去检测哪些文件描述符有可读数据，可写，错误发生。

代码思路：
可以使用发生事件的总数进行控制，减少循环次数
调用 select 涉及到了用户空间和内核空间数值交互过程
事件一共包括两个部分，一类是新连接事件，一类是有数据可读事件

- select 优点：
  - 一个进程可以支持多个客户端
  - select 支持跨平台
- select 缺点：
  - 代码编写困难
  - 会涉及到用户区和内核区的来回拷贝
  - 当客户端多个连接，但少数活跃的情况，select 效率较低 <br> 利润也：作为极端的一种情况，3 - 1023 有发送数据，select 就显得效率低下
  - 最大支持 1024 个客户端连接 <br> select 最大支持 1024 个客户端连接不是有文件描述符最多可以支持 1024 个文件描述符限制的，而是 `FD_SETSIZE=1024` 限制的