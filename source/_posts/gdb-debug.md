---
title: gdb调试
date: 2025-03-07 11:24:38
categories:
- 技术
tags:
- Linux
- gcc
---

# gdb 调试

## gdb 介绍

GBD (DBU Debugger) 是 GCC 的调试工具。其功能强大，现描述如下：

GDB 主要帮忙你完成下面四个方面的功能：

- 启动程序，可以按照你的自定义的要求随心所欲的运行程序。
- 可让被调试的程序在你所指的断点处听出。(断点可以是条件表达式)
- 当程序被停住时，可以检查此时你的程序中所发生的事。
- 动态的改变你程序的执行环境。

## 生成调试信息

一般来说 GDB 主要调试的是 C/C++ 的程序。要调试 C/C++ 的程序，首先在编译时，我们必须要把调试信息加到可执行文件中。使用编译器 (cc/gcc/g++) 的 -g 参数可以做到这一点。如：

```
gcc -g hello.c -o hello
```

如果没有 -g，你将看不到程序的函数名、变量名，所代替的全是运行时的内存地址。当你用 -g 把调试信息加入之后，并成功编译目标代码以后，让我们来看看如何用 gbd 调试它。

## 启动 gdb

- 启动 gdb：`gdb program`
  - program 也就是你的大执行文件，一般在当前目录下。

- 设置运行参数
  - `set args` 可以指定运行时参数。(如：set args 10 20 30 40 50)
  - `show args` 命令可以查看设置好的运行参数

- 启动程序
  - `run`：程序开始执行，如果有断电，停在第一个断点处
  - `start`：程序向下执行一行。(在第一条语句处停止)

## 显示源代码

GBD 可以打印出所调试程序的源代码，当然，在程序编译时一定要加上 -g 参数，把源程序信息编译到执行文件中华。不然就看不到源程序了。当程序停下来后，GDB 会报告程序停在了哪个文件的第几行上。你可以用 list 命令来打印程序的源代码，默认打印 10 行，list命令的用法如下所示：

- `list linenum`：打印第 linenum 行的上下文内容
- `list function`：显示函数名为 function 的函数的源程序
- `list`：显示当前行后面的源程序
- `list -`：显示当前文件开始处的源程序
- `list file:linenum`：显示 file 文件下第 linenum 行
- `list file:function`：显示 file 文件的函数名为 function 的函数的源程序

一般时打印当前行上 5 行和下 5 行，如果显示函数时上 2 行下 8 行，默认是 10 行，当然，你也可以定制显示范围，使用下面命令可以设置一次显示源程序的行数。

- `set listsize count`：设置一次显示源代码的行数
- `show listsize`：查看当前 listsize 的设置

## 设置断点

### 简单断点 -- 当前文件

- break 设置断点，可以简写为 b
  - `b 10`设置断点，在源程序第 10 行
  - `b func`设置断点，在 func 函数入口处

### 多文件设置断点 -- 其他文件

- 在进入指定函数时停住：
  - `b filename:linenum` -- 在源文件 filename 的 linenum 行处停住
  - `b filename:function` -- 在源文件 filename 的 function 函数的入口处停住
- 查询所有断点
  - `info b == info break == i break == i b`
- 条件断点
    
    一般来说，为断点设置一个条件，我们使用 if 关键字，后面跟其断点条件。设置一个条件断点：
    - `b test.c:8 if intValue == 5`
- 维护断点
  - `delete [range...]`删除指定的断点，其简写命令为 d。
    - 如果不指定断点号，则表示删除所有的断点。range 表示断点号的范围
      - 删除某个断点：`delete num`
      - 删除多个断点：`delete num1 num2`
      - 删除连续多个断点：`delete m-n`
      - 删除所有断点：`delete`
    - 比删除更高的一种方法是 disable 停止点，disable 了的停止点，GDB 不会删除，当你还需要时，enable 即可，就好像回收站一样。
  - `disable/enable [range...]` 使指定断点无效，简写命令使 dis/ena。

    如果什么都不指定，表示 diable 所有的停止点。
    - 使一个断点无效/有效：`disable/enable num`
    - 使多个断点无效/有效：`disable/enable num1 num2`
    - 使多个连续的断点无效/有效：`disbale/enable m-n`
    - 使所有断点无效/有效:`disable/enable`

## 调试代码

- `run` 运行程序，可简写为 r
- `next` 单步跟踪，函数调用当作一条简单执行语句执行，可简写为 n
- `step` 单步跟踪，函数调用会进入被调用函数体内，可简写为 s
- `finish` 退出进入的函数，如果出不去，看一下函数体中的循环中是否由断点，如果有删掉，或者设置无效。
- `until` 在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体，可简写为 u，如果出不去，看一下函数体中的循环中是否由断点，如果有删掉，或者设置无效。
- `continue` 继续运行程序，可简写为 c (若有断点，则跳到下一个断点处)

## 查看变量的值

### 查看运行时变量的值

print 打印变量、字符串、表达式等的值，可简写为 p

- `p count` ---- 打印 count 的值

### 自动显示变量的值

你可以设置一些自动显示的变量，当程序停住时，或是在你单步跟踪时，这些变量会自动显示。相关的 GDB 命令时 display。

- `display 变量名`
- `info display` -- 查看 display 设置的自动显示的信息。
- `undisplay num` (info display 时显示的编号)
- `delete display dnums...` -- 删除自动显示，dnums 意为所设置好了的自动显示的编号。如果要同时删除几个，编号可以用空格分隔，如果要删除一个范围内的编号，可以用减号表示
  - 删除某个自动显示：`undisplay num` 或者 ``delete display num`
  - 删除多个: `delete display num1 num2`
  - 删除一个范围：`delete display m-n`
- `disable/enable display dnums`
  - 使一个自动显示无效/有效：`disable/enable display num`
  - 使多个自动显示无效/有效: `disable/enable display num1 num2`
  - 使一个范围的自动显示无效/有效：`disable/enable display m-n`

### 查看修改变量的值

- `ptype width` -- 查看变量 width 的类型
  - `type == double`
- `p width` -- 打印变量 width 的值
  - `$4 = 13`
- 你可以使用 `set var` 命令来告诉 GDB，width 不是你 GDB 的参数，而是程序的变量名，如：
  - `set var width = 47` -- 将变量 var 的值设置为 47
- 在你改变程序变量取值时，最好都是用 set var 格式的 GDB 命令