---
title: 网络编程 (一)
date: 2025-04-08 14:47:45
categories:
- 技术
tags:
- Linux
- C/C++
- 网络编程
---

# 网络编程 (一)

## 网络基础概念

### 协议

概念：协议事先约定好，打架共同遵守的一组规则，如交通信号灯。从应用程序的角度来看，协议可理解为数据传输喝数据解释的规则；可以简单的理解为各个主机之间通信所使用的语言。

假设，A、B 双方欲传输文件。规定：
第一次：传输文件名，接收方接受到文件名，应答 OK 给传输方；
第二次：发送文件的尺寸，接收方收到数据再次应答一个 OK；
第三次：传输文件内容。同样，接收方接收数据完成后应答 OK 表示文件内容接收成功。
![](image1.png)

这种在 A 和 B 之间被遵守的协议称之为原始协议，后来经过不断增加完善改进，最终形成了一个稳定的完整的传输协议，被广泛用于各种文件传输，该协议逐渐就成了一个标准协议。

### 分层模型

OSI 是 Open System Interconnection 的缩写，意为开放式系统互联，国际标注化祖师 (ISO) 制定了 OSI 模型，该模型定义了不同计算机互联的标准，是设计和描述计算机网络通信的基本框架。

网络分层 OSI 7 层模型：
- 物理层 --- 双绞线，光纤 (传输介质)，将模拟信号转换为数字信号
- 数据链路层 --- 数据校验，定义了网络传输的基本单位-帧
- 网络层 --- 定义网络，两台机器之间传输的路径选择点到点的传输
- 传输层 --- 传输数据 TCP，UDP，端到端的传输
- 会话层 --- 通过传输层建立数据传输的通道
- 表示层 --- 编解码，翻译工作
- 应用层 --- 为客户提供各种应用服务，email 服务，ftp 服务，ssh 服务

![](image2.png) ![](image3.png) ![](image4.png)

### 数据通信过程

通信过程：其实就是发送端层层打包，接收方层层解包
注意：这些操作不是用户自己做的，而是底层帮我们做好的。
![](image5.png) ![](image6.png)

### 网络应用程序的设计模式

#### CS 设计模式优缺点

- 优点：
  - 客户端在本机上可以保证性能，可以将数据缓存到本地，提高数据的传输效率，提高用户体验效果
  - 客户端和服务端程序都是由同一个团队开发，协议选择比较灵活
- 缺点：
  - 服务器和客户端都需要开发，工作量相对较大，调试困难，开发周期长
  - 从用户的角度看，需要将客户端安装到用户的主机上，对用户主机的安全构成威胁

#### BS 设计模式优缺点

- 优点：
  - 无需安装客户端，可以使用标准的浏览器作为客户端
  - 只需要开发服务器，工作量相对较小
  - 由于采用标准客户端，所以移植性好，不受平台限制
  - 相对安全，不用安装软件
- 缺点：
  - 由于没有客户端，数据缓冲不尽人意，数据传输有限制，用户体验较差
  - 通信协议选择只能使用 HTTP 协议，协议选择不够灵活

### 以太帧格式

以太帧格式就是包装在网络接口层 (数据链路层) 的协议
![](image7.png)

以 APR 为例介绍以太网帧格式
![](image8.png)
目的端 mac 地址是通过发送端发送 ARP 广播，接收到该 ARP 数据的主机先判断是否是自己的 IP，若是则应答一个 ARP 应答报文，并将 mac 地址填入应答报文中；若目的 IP 不是自己的主机，则直接丢弃该 ARP 请求报文。

#### IP 格式段

![](image9.png)
- 协议版本：IPv4，IPv6
- 16 位总长度：最大 65536
- 8 位生存时间 ttl (网络连接到下一跳的次数)：为了防止网络阻塞
- 32 位源 IP 地址，共 4 个字节！我们熟悉的 IP 都是点分十进制，4 字节，每字节对应一个点分- 位，最大为 255，实际上就是整型数。
- 32 位目的 IP 地址
- 8 位协议：用来区分上层协议是 TCP，UDP，ICMP 还是 IGMP 协议。
- 16 位首部校验和：只校验 IP 首部，数据的校验是由更高层协议负责

#### UDP 数据报格式

![](image10.png)

通过 IP 地址来确定网络环境中的唯一的一台主机；
主机上使用端口号来区分不同的应用程序。
IP + 端口唯一确定唯一一台主机上的一个应用程序

#### TCP 数据流格式

![](image11.png)

- 序号：TCP 是安全可靠的，每个数据包都带有序号，当数据包丢失的时候，需要重传，要使用序号进行重传，控制数据有序，丢包重拾。
- 确认序号：使用确认序号可以知道对方是否已经收到了，通过确认序号可以知道哪个序号的数据需要重传
- 16 位窗口大小 -- 滑动窗口 (主要进行流量控制)

## SOCKET 编程

传统的进程间通信借助内核提供的 IPC 机制进行，但是只能限于本机通信，若要跨机通信，就必须使用网络通信。(本质上借助内核 - 内核提供了 socket 伪文件机制实现通信 --- 实际上是使用文件描述符)，这就需要使用内核给用户提供的 socket API 函数库

既然提到 socket 伪文件，所以可以使用文件描述符相关的函数 read、write
可以对比 pipe 管道讲述 socket 文件描述符的区别

如下图，一个文件描述符操作两个缓冲区，这点跟管道是不同的，管道是两个文件描述符操作一个内核缓冲区。

![](image14.png)

### socket 编程预备知识

- 网络字节序：
  - 大端：低位地址存放高位数据，高位地址存放低位数据
  - 小端：低位地址存放低位数据，高位地址存放高位数据
- 大端和小端的使用场合
  - 大端和小端只是对数据类型长度是两个及以上的，如 int、short，对于单字节没限制，在网络中经常考虑大端和小端的是 IP 和端口。

大小端验证程序：
```c
#include <stdio.h>

int
isLittleEndian()
{
  int num = 0x01020304;
  char* p = (char*)&num;
  if (*p == 0x04) {
    return 1;
  }
  return 0;
}

int
main()
{
  if (isLittleEndian()) {
    printf("Little Endian\n");
  } else {
    printf("Big Endian\n");
  }
  return 0;
}
```

网络传输用的是大端法，如果机器用的是小端法，则需要进行大小端转换
下面 4 个函数就是进行大小端转换的函数：

```c
#include <arpa/inet.h>
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

函数名的 h 表示主机 host，n 表示网络 network，s 表示 short，l 表示 long
上述的几个函数如果本来不需要转换函数就不会做转换

#### IP 地址转换函数

p -> 表示点分十进制的字符串形式
to -> 到
n -> 表示 network 网路

- `int inet_pton(int af, const char *src, void *dst);`
  - 函数说明：将字符串形式的点分十进制 IP 转换为大端模式的网络 IP (整形 4 字节数)
  - 参数说明：
    - af：AF_INET
    - src：字符串形式的点分十进制 IP 
    - dst：存放转换后的变量的地址

手工也可以计算：
如 192.168.232.145, 先将4个正数分别转换为16进制数,
192 --> 0xC0  168 --> 0xA8   232 --> 0xE8   145 --> 0x91
最后按照大端字节序存放: 0x91E8A8C0，这个就是4字节的整形值

- `const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);`
  - 函数说明：网络IP转换为字符串形式的点分十进制的IP
  - 参数说明：
    - af：AF_INET
    - src：网络的整形的 IP 地址
    - dst：转换后的 IP 地址,一般为字符串数组
    - size：dst 的长度
    - 返回值：
      - 成功 -- 返回指向 dst 的指针
      - 失败 -- 返回 NULL，并设置 errno

例如：
IP 地址为 010aa8c0，转换为点分十进制的格式：
01 --> 1    0a --> 10   a8 --> 168   c0 --> 192
由于从网络中的 IP 地址是高端模式, 所以转换为点分十进制后应该为：192.168.10.1

#### 结构体 struct sockaddr

![](image15.png)

```c
// sockaddr 结构
struct sockaddr {
    sa_family_t sa_family;
    char     sa_data[14];
};

// sockaddr_in 结构
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};

/* Internet address. */
struct in_addr {
    uint32_t  s_addr;     /* address in network byte order */
};//网络字节序IP--大端模式
```

可通过 `man 7 ip` 可以查看相关说明

### socket 变成主要的 API 函数介绍

- 创建socket
  - 函数原型：`int socket(int domain, int type, int protocol);`
  - 参数说明：
    - domain：协议版本
      - AF_INET -- IPv4
      - AF_INET6 -- IPv6
      - AF_UNIX AF_LOCAL 本地套接字使用
    - type：协议类型
      - SOCK_STREAM 流式，默认使用的协议是 TCP 协议
      - SOCK_DGRAM 报式，默认使用的协议是 UDP 协议
    - protocol：一般填 0，表示使用对应类型的默认协议
  - 返回值：
    - 成功：返回一个大于 0 的文件描述符
    - 失败：返回 -1，并设置 errno

当调用 socket 函数以后，返回一个文件描述符，内核会提供与该文件描述符对用的读和写缓冲区，同时还有两个队列，分别是请求连接队列和已连接队列

![](image14.png)

- 绑定套接字
  - 函数原型：`int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`
  - 参数说明：
    - socket：调用 socket 函数返回的文件描述符
    - addr：本地服务器的 IP 地址和 PORT
    - addrlen：addr 变量的占用内存大小
  - 返回值：
    - 成功返回 0，失败返回 -1，并设置 errno

- 监听套接字
  - 函数原型：`int listen(int sockfd, int backlog);`
  - 参数说明：
    - socket：调用 socket 函数返回的文件描述符
    - backlog：同时请求连接的最大个数 (还未建立连接)
  - 返回值：
    - 成功返回 0，失败返回 -1，并设置 errno

- 接收连接
  - 函数原型：`int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);	`
  - 函数参数：
    - sockfd：调用 socket 函数返回的文件描述符
    - addr：传出参数，保存客户端的地址信息
    - addrlen：传入参数，addr 变量所占内存空间大小
  - 返回值：
    - 成功：返回一个新的文件描述符，用于和客户端通信
    - 失败：返回 -1，并设置 errno

accept 函数是一个阻塞函数，若没有新的连接请求，则一直阻塞。
从已连接队列中获取一个新的连接，并获得一个新的文件描述符，该文件描述符用于和客户端通信。(内核会负责将请求队列中的连接拿到已连接队列中)

- 连接服务器
  - 函数原型：`int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`
  - 函数参数：
    - sockfd：调用 socket 函数返回的文件描述符
    - addr：服务器的地址信息
    - addrlen：addr 变量的内存大小
  - 返回值：成功返回 0，失败返回 -1，并设置 errno

接下来就是读取和发送数据了
```c
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
// 对应 recv 和 send 这两个函数 flags 直接填 0 就可以了。
```

使用 socket 的 API 函数编写服务端和客户端程序的步骤图示：
![](image16.png)