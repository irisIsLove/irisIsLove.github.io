---
title: 文件IO
date: 2025-03-21 15:49:41
categories:
- 技术
tags:
- Linux
- C/C++
---

# 文件 IO

## C 库 IO 函数的工作流程

![](c_library_io_function_workflow1.png)

![](c_library_io_function_workflow2.png)

c 语言操作文件相关问题：

使用 fopen 函数打开一个文件，返回一个 FILE* fp，这个指针指向的结构体有三个重要的成员。

- 文件描述符：通过文件描述符可以找到文件的 inode，通过 inode 可以找到对应的数据块
- 文件指针：读和写共享一个文件指针，读或者写都会引起文件指针的变化。
- 文件缓冲区：读或者写会先通过文件缓冲区，主要目的是为了减少对磁盘的读写次数，提高读写磁盘的效率。

备注：
- 头文件 stdio.h 的第 48 行处：`typedef struct_IO_FILE FILE`;
- 头文件 libio.h 的第 241 行处：`struct_IO_FILE`，这个接头文件定义中有一个 `_fileno_`成员，这个就是文件描述符。

## C 库函数与系统函数的关系

![](c_function_with_system_function.png)

系统调用：由操作系统实现并提供给外部应用程序的编程接口，(Application Programming Interfact, API)，是应用程序与系统之间数据交互的桥梁。

## 虚拟地址空间

![](virtual_address.png)

进程的虚拟地址空间分为用户区和内核区，其中内核区是受保护的，用户是不能够对其进行读写操作的。

内核区中很重要的一个就是进程管理，进程管理中有一个区域就是 PCB(本质是一个结构体)。

PCB 中有文件描述符表，文件描述符表中存放着打开的文件描述符，涉及到文件的 IO 操作都会用到这个文件描述符。

## PCB 和文件描述符表

![](pdb_and_file_descriptor_table.png)

备注：

pcb：结构体：`task_struct`，该结构体在：`/usr/src/linux-headers-4.4.0-97/include/linux/sched.h:1390`

一个进程有一个文件描述符表：1024
- 前三个被占用，分别是`STDIN_FILENO`，`STDOUT_FILENO`，`STDERR_FILENO`
- 文件描述符作用：通过文件描述符找到 inode，通过 inode 找到磁盘数据块

虚拟地址空间 -> 内核区 -> PCB -> 文件描述符表 -> 文件描述符 -> 文件 IO 操作使用文件描述符

## open/close

### 文件描述符

一个进程启动之后，默认打开三个文件描述符：
```c
#define STDIO_FILENO    0
#define STDOUT_FILENO   1
#define STDERR_FILENO   2
```

新打开文件返回文件描述符中未使用的最小文件描述符，调用 open 函数可以打开或创建一个问及那，得到一个文件描述符。

### open 函数

- 函数描述：打开或者新建一个文件
- 函数原型：
  - `int open(const char* pathname, int flags);`
  - `int open(const char* pathname, int flags, mode_t mode);`
- 函数参数：
  - pathname 参数是要开打或创建的文件名，和 fopen 一样，pathname 既可以是相对路劲也可以是绝对路径
  - flags 参数有一系列常数值可供选择，可以同时选择多个常熟用按位或运算连接起来，所以这些常熟的共定义都已 O_ 开头，表示 or。
    - 必选项：以下三个常数中必须指定一个，且仅允许指定一个。
      - `O_RDONLY` 只读打开
      - `O_WROBLY` 只写打开
      - `O_RDWR` 可读可写打开
    - 以下可选项可以同时指定 0 个或多个，和必选项按位或起来作为 flags 参数。以下为常用项：
      - `O_APPEND` 表示追加。如果文件已有内容，这次打开文件所写的数据附加到文件的末尾而不是覆盖原来的内容。
      - `O_CERAT` 若此文件不存在则创建它。使用此选项时需要提供第三个参数 mode，表示该文件的访问权限。
        - 文件最终权限：`mode&~umask`
      - `O_EXCL` 如果同时指定了 O_CREAT，并且文件已存在，则出错返回
      - `O_TRUNC` 如果文件已存在，将其长度截断为 0 字节
      - `O_NONBLOCK` 对于设备文件，以 O_NONBLOCK 方式打开可以做非阻塞 I/O(Nonblock I/O)
- 函数返回值：
  - 成功：返回一个最小且未被占用的文件描述符
  - 失败：返回 -1，并设置 errno 值

### close 函数

- 函数描述：关闭文件
- 函数原型：`int close(int fd);`
- 函数参数：fd 文件描述符
- 函数返回值：
  - 成功返回 0
  - 失败返回 -1，并设置 errno 值

需要说明的是，当一个进程终止时，内核对该进程所有尚未关闭的文件描述符调用 close 关闭，所以即使用户程序不调用 close，在终止时也会自动关闭它打开的所有文件。但是对于一个长年累月云心过的程序（比如网络服务器），打开的文件描述符一定要记得关闭，否则随着打开的文件越来越多，会占用大量文件描述符和系统资源。

## read/write

### read 函数

- 函数描述：从打开的设备或文件中读取数据
- 函数原型：`ssize_t read(int fd, void *buf, size_t count);`
- 函数参数：
  - fd：文件描述符
  - buf：读上来的数据保存在缓冲区 buf 中
  - count：buf 缓冲区存放的最大字节数
- 函数返回值
  - \>0：读取到的字节数
  - =0：文件读取完毕
  - -1：出错，并设置 errno

### write 函数

- 函数描述：向打开的设备或文件中写数据
- 函数原型：` ssize_t write(int fd, const void *buf, size_t count);`
- 函数参数：
  - fd：文件描述符
  - buf：缓冲区，要写入文件或设备的数据
  - count：buf 中数据的长度
- 函数返回值：
  - 成功：返回写入的字节数
  - 错误：返回 -1 并设置 errno

## lseek

所有打开的文件都有一个当前文件偏移量(current file offset)，一下简称为 cfo。cfo通常是一个非负整数，用于表明文件开始处到文件当前位置的字节数。读写操作通常开始于 cfo，并且使 cfo 增大，增量为读写的字节数，文件被打开时，cfo 会被初始化为 0，除非使用了 O_APPEND。

使用 lseek 函数可以改变文件的 cfo。

```c
#include <sys/types.h>
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
```

- 函数描述：移动文件指针
- 函数原型：`off_t lseek(int fd, off_t offset, int whence);`
- 函数参数：
  - fd：文件描述符
  - 参数 offset的含义取决于参数 whence
    - 如果 whence 是 SEEK_SET，文件偏移量将设置为 offset
    - 如果 whence 是 SEEK_CUR，文件偏移量将被设置为 cfo 加上 offset，offset 可以为正也可以为负
    - 如果 whence 是 SEEK_END，文件偏移量将被设置为文件长度加上 offset，offset 可以为正也可以为负
- 函数返回值：若 lseek 成功执行，则返回新的偏移量
- lseek 函数常用操作：
  - 文件指针移动到头部：`lseek(fd, 0, SEEK_SET)`
  - 获取文件指针当前位置：`int len = lseek(fd, 0, SEEK_CUR)`
  - 获取文件长度：`int len = lseek(fd, 0, SEEK_END)`
  - lseek 实现文件拓展：
    ```c
    // 从文件尾部开始向后拓展 1000 个字节
    off_t curpos = lseek(fd, 1000, SEEK_END);
    // 额外执行一次写操作，否则文件无法完成拓展
    write(fd, "a", 1);
    ```