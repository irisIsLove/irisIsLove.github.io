---
title: makefile
date: 2025-03-06 16:33:15
categories:
- 技术
tags:
- Linux
- gcc
---

# makefile

makefile 文件中定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 makefile 就像一个 shell 脚本一样，其中也可以执行操作系统的命令。makefile 带来的好处就是 --- "自动化编译"，一旦写好，只需要一个 make 命令，整个工程完全自动编译，极大的提高了软件开发的效率。

make 是一个命令工具，是一个解释 makefile 中指令的命令工具，一般来说，大多数的 IDE 都有这个命令，比如：Visual C++ 的 nmake，Linux 下的 make。可见，makefile 都成为了一种在工程方面的编译方法。

makefile 文件中会使用 gcc 编译器对源代码进行编译，最终生成可执行文件或者库文件。

makefile 文件的命名：makefile 或者 Makefile

## makefile 的基本规则

makefile 由一组规则组成，规则如下：

```
目标：依赖
(Tab) 命令
```

makefile 基本规则三要素：
- 目标：要生成的目标文件
- 依赖：目标文件由哪些文件生成
- 命令：通过执行该命令由依赖文件生成目标

下面以具体的例子讲解：

当前目录下由 main.c fun1.c fun2.c sum.c，根据这个基本规则编写一个简单的 makefile文件，生成可执行文件 main。

第一个版本的 makefile 文件：
```makefile
main: main.c fun1.c fun2.c sum.c
	gcc -o main main.c fun1.c fun2.c sum.cls 
```
缺点：效率低，修改一个文件，所有的文件会全部重新编译。

## makfile 工作原理

### 基本原则：

**若想生成目标，检查规则中的所有的依赖文件是否都存在：**

- 如果有的依赖文件不存在，则向下搜索规则，看是否由生成该依赖文件的规则：

    如果有规则用来生成该依赖文件，则执行规则中的命令生成依赖文件；如果没有规则用来生成该依赖文件，则报错。
    ![](makefile1.png)

- 如果所有依赖都存在，检查规则中的目标是否需要更新，必须先检查它的所有依赖，依赖中有任何一个被更新，则目标必须更新。(检查的规则是哪个时间大，哪个最新)
  - 若目标的时间 > 依赖的时间，不更新
  - 若目标的时间 < 依赖的时间，更新

    ![](makefile2.png)

#### 总结：
- 分析各个目标和依赖之间的关系
- 根据依赖关系自底向上执行命令
- 根据依赖文件得到时间和目标文件的时间确定是否需要更新
- 如果目标不依赖任何条件，则执行对应命令，以示更新(如：伪目标)

第二个版本：
```makefile
main: main.o fun1.o fun2.o sum.o
	gcc -o main main.o fun1.o fun2.o sum.o

main.o: main.c
	gcc -c main.c -I./

fun1.o: fun1.c
	gcc -c fun1.c

fun2.o: fun2.c
	gcc -c fun2.c

sum.o: sum.c
	gcc -c sum.c
```
缺点：冗余，若 .c 文件数量很多，编写起来比较麻烦

## makefile 中的变量

在 makefile 中使用变量有点类似于 C 语言中的宏定义，使用该变量相当于内容替换，使用变量可以使 makefile 易于维护，修改起来变得简单。

makefile 有三种类型的变量：
- 普通变量
- 自带变量
- 自动变量

### 普通变量

- 变量定义直接用 =
- 使用变量值用 $(变量名)
    ```makefile
    如：下面是变量的定义和使用：
        foo = abc       // 定义变量并赋值
        bar = $(foo)    // 使用变量
    ```
    定义了两个变量：foo、bar，其中 bar 的值是 foo 变量值的引用。

### 自带变量

除了用户自定义变量，makefile 中也提供了一些变量(变量名大写)供用户直接使用，我们可以直接对其进行赋值：

```makefile
CC = gcc #arm-linux-gcc
CPPFLAGS：C 预处理的选项 -I
CFLAGS：C 编译器的选项 -Wall -g -c
LDFLAGS：链接器的选项 -L -l
```

### 自动变量

- $@：表示规则中的目标
- $<：表示规则中的第一个条件
- $^：表示规则中的所有条件，组成一个列表，以空格隔开，如果这个列表中有重复的项则消除重复项。
- 特别注意：自动变量只能在规则的命令中使用

### 规则模式

至少在规则的目标定义中要包含 '%' ，'%' 表示一个或多个，在以来条件中同样可以使用 '%'，依赖条件中的 '%' 的取值却决于其目标：

比如：main.o : main.c fun1.o : fun1.c fun2.o : fun2.c sum.o : sum.c，说的简单点就是：xxx.o : xxx.c

makefile 的第三个版本：
```makefile
target = main
objects = main.o fun1.o fun2.o sum.o
CC = gcc
CPPFLAGS = -I./

$(target) : $(objects)
	$(CC) -o $@ $^

%.o : %.c
	$(CC) -o $@ -c $< $(CPPFLAGS)
```

## makefile 中的函数

makefile 中的函数有很多，在这里给打架介绍两个最常用的。

1. wildcard - 查找指定目录下的指定类型的文件
   
    `src = $(wildcard ./*.c)`     // 找到当前目录下所有后缀为 .c 的文件，赋值给src

2. patsubst - 匹配替换

    `obj = $(patsubst %.c,%.o,$(src))`    // 将 src 中所有后缀为 .c 的文件替换为 .o

在 makefile 中所有的函数都是由返回值的。

当前目录下由 main.c fun1.c fun2.c sum.c

`src = $(wildcard ./*.c)` 等价于 src = main.c fun1.c fun2.c sum.c

`obj = $(patsubst %.c,%.o,$(src))` 等价于 obj = main.o fun1.o fun2.o sum.o

makefile 的第四个版本：
```makefile
src = $(wildcard ./*.c)
objects = $(patsubst %.c, %.o $(src))
target = main
CC = gcc
CPPFLAGS = -I./

$(target) : $(objects)
	$(CC) -o $@ $^

%.o : %.c
	$(CC) -o $@ -c $< $(CPPFLAGS)
```

缺点：每次重新编译都需要手工清理中间文件和最终目标文件

## makefile 的清理操作

用途：清楚编译生成的中间文件和最终目标文件。

make clean 如果当前目录下由同名 clean 文件，则不执行 clean 对应的命令，解决方案：
- 伪目标声明：
  
  - `.PHONY : clean`
  - 声明目标为伪目标之后，makefile 将不会检查该目标是否存在或者该目标是否需要更新。

- clean 命令中的特殊符号：
  - "-" 此命令出错，make 也会继续执行后续的命令。如：`-rm main.o`

    rm -f：将之执行，比如若要删除的文件不存在使用 -f 不会报错

- 其他：
  - make 默认执行第一个出现的目标，可通过 make dest 指定要执行的目标
  - make -f：-f执行第一个 makefile 文件名称，使用 make 执行指定的 makefile：make -f mainmak

makefile 的第五个版本：
```makefile
src = $(wildcard ./*.c)
objects = $(patsubst %.c, %.o, $(src))
target = main
CC = gcc
CPPFLAGS = -I./

$(target) : $(objects)
	$(CC) -o $@ $^

%.o : %.c
	$(CC) -o $@ -c $< $(CPPFLAGS)

.PHONY : clean
clean:
	rm -f $(objects) $(target)
```

在 makefile 的第五个版本中，综合使用了变量，函数，规则模式和清理命令，是一个比较完善的版本。