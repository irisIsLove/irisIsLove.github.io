---
title: 文件IO(原理)
date: 2025-03-21 15:49:41
categories:
- 技术
tags:
- Linux
- C/C++
---

# 文件 IO (原理)

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