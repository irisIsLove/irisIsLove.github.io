---
title: gcc编译器
date: 2025-03-04 15:40:20
categories:
- 技术
tags:
- Linux
- gcc
---

# gcc 编译器

## gcc 的工作流程

gcc 编译器将 c 源文件到生成一个可执行程序，中间一共经历了四个步骤：
![](gcc_compile_flow.png)

四个步骤并不是 gcc 独立完成的，而是在内部调用了其他工具，从而完成了整个工作流程，其中编译最耗时，因为要逐行检查语法。
![](gcc_tool_chain.png)

下面以 test.c 为例介绍 gcc 的四个步骤：
```shell
gcc -E test.c -o test.i
gcc -S test.i -o test.s
gcc -c test.s -o test.o
gcc test.o -o test
```

一步生成最终可执行程序：
```shell
gcc test.c -o test
```

## gcc 常用参数

- -v 查看 gcc 版本号， --version 也可以
- -E 生成预处理文件
- -S 生成汇编文件
- -c 只编译，生成 .o 文件，通常称为目标文件
- -I 指定头文件所在的路径
- -L 指定库文件所在的路径
- -l 指定库的名字
- -o 指定生成的目标文件的名字
- -g 包含调试信息，使用 gdb 调试需要添加 -g 参数
- -On n=0~3 编译优化，n 越大优化程度越高，但编译时间越长

例如：下面代码片段
```c
int a = 10;
int b = a;
int c = b;
printf("%d", c);
```

上面的代码可能会被编译器优化成：
```c
int c = 10;
printf("%d", 10);
```

- -Wall 提示更多警告信息

```c
int a;
int b;
int c = 10;
printf("[%d]\n", c);
```
编译如下：

```shell
gcc -o test -Wall test.c
warning: unused variable 'a' [-Wunused-variable]
warning: unused variable 'b' [-Wunused-variable]
```

- -D 编译时定义宏
test.c 文件中的代码片段：
```c
printf("max==[%d]", MAX);
```

编译：
```shell
gcc -o test test.c -D MAX=10
gcc -o test test.c -DMAX=10
```