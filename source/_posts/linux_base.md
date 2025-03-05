---
title: Linux基础命令
date: 2025-03-03 10:01:02
categories:
- 技术
tags:
- Linux
---

# Linux基础命令

## Shell 相关
    概念： Shell就是命令解析器，Shell将用户输入的命令解释成内核能够识别的指令，Shell就相当于翻译。
    查看当前系统使用的shell:
        echo $SHELL
    查看当前系统支持的shell:
        cat /etc/shells

## Linux目录结构
    是一个倒立的树状结构。
- /bin: binary，二进制文件，可执行程序，shell命令
  - 如：ls, rm, mv, cp等常用命令
- /sbin: s是Super User的意思，这里存放的是系统管理员使用的系统管理程序。
  - 如ifconfig, halt, shutdown, reboot等系统命令
- /dev: device，在linux下一切皆文件
  - 硬盘, 显卡, 显示器
  - 字符设备文件、块设备文件
    - 如: 在input目录下执行: sudo cat mouse0, 移动鼠标会显示有输入.
- /lib: linux运行的时候需要加载的一些动态库
  - 如: libc.so、libpthread.so等
- /mnt: 手动的挂载目录, 如U盘等
- /media: 外设的自动挂载目录, 如光驱等
- /root: linux的超级用户root的家目录
- /usr: unix system resource--类似于WINDOWS的programe files目录
  - include目录里存放头文件, 如: stdio.h、stdlib.h、string.h、pthread.h
  - games目录下的小游戏-如: sl小火车游戏
- /etc: 存放配置文件
  - /etc/passwd
    - man 5 passwd可以查看passwd文件的格式信息
  - /etc/group
    - man 5 group可以查看group文件的格式信息
  - /etc/profile
    - 系统的配置文件, 修改该文件会影响这个系统下面的所有的用户
- /opt: 安装第三方应用程序
  - 比如安装oracle数据库可以在这个目录下
- /home: linux操作系统所有用户的家目录
  - 用户家目录：(宿主目录或者主目录)
    - /home/itcast
- /tmp: 存放临时文件
  - 新建在这个目录下的文件会在系统重启后自动清除

## 相对路径和绝对路径
  - 绝对路径: 从/目录开始表示的目录.
  - 相对路径: 从当前目录开始表示的目录.
    - 从当前所处的目录开始表示的路径。
    - . 表示当前目录
    - .. 表示当前目录的上一级目录

## 文件和目录操作相关的命令

### tree 命令
    以树状星蚀查看指定目录内容，使用该命令需要安装软件tree
命令使用方法：
- tree -- 树状结构显示当前目录下的文件信息
- tree 目录 -- 树状结构显示指定目录下的文件信息
  
### ls 命令
- 查看指定目录下的文件信息
- 使用方法：
  - ls -- 显示当前目录下的文件信息
  - ls 目录 -- 显示指定目录下的文件信息
- 相关参数
  - -a: 显示所有文件和目录
    - . 当前目录
    - .. 当前目录的额上一级目录
    - 隐藏文件，以 . 开头的文件，如.bashrc
    - 普通文件
  - -R: 递归显示指定目录下的文件信息
  - -l: 列出文件的详细信息，7部分内容
![](ls-l.png)
  - 参数之间可以结合使用：
    - ls -la: 列出当前目录下所有文件的相关信息，包括隐藏文件
    - ls -ltr: 列出当前目录下的文件，按照时间你想排序

### cd 命令
    切换目录(change directory)，命令使用方式：cd + 目录
    路径可以使用相对路径或者绝对路径
- 切换到家目录
  - cd
  - cd ~
  - cd /home/user_name
  - cd $HOME
- 临近两个目录切换
  - cd -

### pwd 命令
    查看用户当前所处的工作目录，print working directory

### which 命令
    显示命令所在的目录, 如 which ls， which cp

### touch 命令
    如果文件不存在，创建新文件，如果文件存在，更新最后修改时间
    命令使用方式：touch 文件名

### mkdir 命令
    创建新目录，make directory
    创建方式：mkdir 目录名
    如果创建多级目录需要添加参数 -p：
      在当前目录下创建目录： mkdir mydir
      在宿主目录下创建多级目录： mkdir -p ~/test/hello/world/aa

### rmdir 命令
    删除空目录，只能删除空目录 ，使用方式：rmdir 目录名

### rm 命令
- 删除文件： rm 文件名
- 删除目录： rm -r 目录名
- 参数：
  - -r： 递归删除目录，删除目录碧玺添加此参数
  - -i： 提示用户是否删除文件或目录
  - -f： 强制删除
- 注意事项：
  - 使用 rm 命令删除的文件或目录不会放入回收站中，数据不易回复。

### cp 命令
命令使用方式： cp 源目录或文件 目标目录或文件

若有目录的拷贝需要使用 -r 参数
- cp 要拷贝的文件(file1) file(不存在)
  - 创建 file，将 file1 中的内容拷贝到 file
- cp file1 file(存在)
  - file1 覆盖 file
- cp file dir(存在)
  - 拷贝 file 到 dir 目录
- cp -r dir(存在) dir1(存在)
  - 将 dir 目录拷贝到 dir1 目录中
  - 包括 dir 目录
- cp -r dir(存在) dir1(不存在)
  - 创建 dir1
  - 将 dir 目录拷贝到 dir1 中，不包括 dir 目录
- cp 拷贝目录也可以用 -a 参数，这样可以保留被拷贝的文件的一些属性信息

### mv 命令
- 改名或者移动文件 mv file1 file2
  - 改名：
    - mv file(存在) file1(不存在)
    - mv dir(存在) dir1(不存在)
    - mv file(存在) file1(存在)
      - file 文件覆盖 file1 文件，file 改名为 file1
  - 移动(第二个参数一定是目录文件)
    - mv file(文件) dir(存在目录)
      - 将 file 文件移动到 dir 中
    - mv dir(目录) dir1(存在目录)
      - 将 dir 目录移动到 dir1 中，dir 就会作为 dir1 的子目录而存在

### cat 命令
- 将文件内容一次性输出到终端。
- 使用方式： cat 文件名
- 缺点：终端显示的内容有限，如果文件太长无法显示全部。
- 可用于文件重定向：cat file1 > file2， 相当于 cp file1 file2

### more 命令
- 文件内容分页显示到终端，但是智能一直向下浏览，不能回退
- 使用方式：more + 文件名
- 相关操作：
  - 显示下一行：回车
  - 显示下一页：空格
  - 退出：q(ctrl + c)

### less 命令
- 文件内容分页显示到终端，可以上下浏览
- 使用方式：less + 文件名
- 相关操作：
  - 显示下一行：回车、ctrl + p、方向键上
  - 显示上一行：ctrl + n、方向键下
  - 显示下一页：空格、PageDown
  - 显示上一页：PageUp
  - 退出：q

### head 命令
- 从文件头部开始查看前 n 行的内容
- 使用方式： head -n\[行数\] 文件名
  - head -20 hello.txt'
- 如果没有指定行数，默认显示前10行内容

### tail 命令
- 从文件尾部向上查看最后 n 行的内容
- 使用方式： tail -n\[行数\] 文件名
- 如果未指定行数，默认显示最后10行内容
- 一个比较重要的应用：显示日志：`tail -f test.log`，可以实时显示日志内容，当有新的日志产生时，会自动显示在终端上

### 软链接
- 软链接类似于 windows 下的快捷方式
- 如何创建软链接
  - ln -s 文件名 快捷方式的名字
        
    例如：ln -s aa aa.soft
  - 目录也可以创建软链接

    例如：ln -s tmp tmp.link
- 创建软链接应注意事项
  - ln 创建软链接要用绝对路径，因为如果不使用绝对路径，一旦这个链接文件发生位置变动，就不能找到那个文件了。
  - 软链接文件的大小是：路径 + 文件名的总字节数

### 硬链接
- ln 文件名 硬链接的名字
  - ln test.log test.log.hard
- 使用硬链接应注意事项
  - 硬链接不能建立在目录上
  - 硬链接对绝对路径没有要求
  - 硬链接不能跨文件系统

    硬链接文件和源文件的 inode 是相同的，文件系统的 inode 要求唯一，跨文件系统可能会使 inode不同，所以硬链接不能跨文件系统
- 硬链接的本质
  - 硬链接的本质使不同的文件名所在 inode 节点是相同的，相同的 inode 节点指向了相同的数据块，所以它们的文件内容是一样的，文件内容会同步。
    - ls -i 文件名 -----> 可以查看文件的 i 节点
    - stat 文件名 -----> 可以查看文件的详细信息，包括 inode 节点
    - 如下图，file.hard 是 file 的硬链接，这两个文件指向同一个 inode，同一个 inode 指向了相同的数据块(文件内容)。
![](hardlink.png)

      - 当新创建了一个文件，硬链接计数为1
      - 给文件创建了一个硬链接后，硬链接计数加1
      - 删除一个硬链接后，硬链接计数减1
      - 如果删除硬链接后，硬链接技术为0，则该文件会被删除
- 硬链接应用场合
  - 可以起到文件同步的作用

    修改 file 内容，会在其余几个硬链接文件上同步
  - 可以起到保护文件的作用

    删除文件的时候，只要硬链接技术不为0，不会真正删除，起到保护文件的作用。

### wc 命令
- 显示文件行数，字节数，单词数
  - wc -l file 显示文件的总行数
  - wc -c file 显示文件的总字节数
  - wc -w file 显示文件的总单词数
  - wc file 显示文件的总行数，总字节数，总单词数

### whoami 命令
- 显示当前用户名

## 用户权限、用户、用户组

### 修改文件权限 chmod
    linux 是通过权限对文件进行控制的，通过使用 chmod 命令可以修改文件相关的权限
- 文字设定法
  - 命令：chmod \[+|-|=\] 文件名
    - 操作对象 \[who\]
      - u -- 用户(user)
      - g -- 同组用户(group)
      - o -- 其他用户(other)
      - a -- 所有用户(all)
    - 操作符 \[\+|-|=\]
      - \+ -- 添加权限
      - \- -- 删除权限
      - = -- 赋予给定权限并取消其他权限
    - 权限 \[mode\]
      - r -- 读权限
      - w -- 写权限
      - x -- 执行权限
  - 示例：给文件 file.txt 的所有者和所属组添加读写权限
    - chmod ug+wr file.txt
- 数字设定法
  - 命令：chmod \[who\] \[+|-|=\] 文件名
    - 操作符 \[\+|-|=\]
      - \ -- 添加权限
      - \- -- 删除权限
      - = -- 赋予给定权限并取消其他权限
    - 数字表示的含义
      - 0 -- 没有权限(-)
      - 1 -- 执行权限(x)
      - 2 -- 写权限(w)
      - 4 -- 读权限(r)
    - 例：给 file.txt 文件设置 rw-rw-r--
      - chmod 664 file.txt
  
注意点：使用数字设定发，一定要使用3位的8进制数：如066

### 修改文件所有者和所属组

- 修改文件所有者 chown
  - 用法：chown 文件所有者 文件名
    - sudo chown user file.txt
- 修改文件所有者和所属组 chown
  - 用法：chown 文件所有者：文件所属组 文件名
    - sudo chown user:group file.txt
- 注意：普通用户需要使用管理员用户权限执行该命令
- 注意：若系统没有其他用法，可以使用 `sudo adduser 用户名` 创建一个新用户

### 修改文件所属组
- chgrp 命令
  - 使用方法：chgrp 用户组 文件或目录名
    - 示例：修改文件所属组位 group1
    - sudo chgrp group1 file.txt
  - 普通用户需要使用管理员用户权限执行该命令

## find 命令

### 按文件名查询：使用参数 -name

- 命令：find 路径 -name "文件名"
- 示例：find /home -name "*.c"

### 按文件类型查询：使用参数 -type

- 命令：find 路径 -type 类型
  - 普通文件类型用 f 表示而不是 -
  - d -> 目录
  - l -> 符号链接
  - b -> 块设备
  - c -> 字符设备
  - s -> socket 文件
  - p -> 管道文件
- 查找指定目录下的普通文件： find 路径 -type f

### 按文件大小查询：使用参数 -size

- 命令：find 路径 -size 范围
  - 范围：
    - 大于：+ 表示 -- +100k
    - 小于：- 表示 -- -100k
    - 等于：无符号表示 -- 100k
  - 大小：
    - M 必须大写(10M)
    - k 必须小写(10k)
    - c 表示字节数
  - 例子：查询目录为家目录
    - 等于 100k 的文件：find ~/ -size 100k
    - 大于 100k 的文件：find ~/ -size +100k
    - 大于 50k，小于 100k 的文件：find ~/ -size +50k -size -100k

### 按文件日期

- 创建日期：-ctime -n/+n
- 修改日期：-mtime -n/+n
- 访问日期：-atime -n/+n

### 按深度

- maxdepth n(层数)
  - 搜索 n 层以下的目录，搜索的层数不超过 n 层
- mindepth n(层数)
  - 搜索 n 层以上的目录，搜索的层数不能小于 n 层

### 高级查找

- 例：查找指定目录下所有目录，并列出目录中文件详细信息
  - find ./ -type d -exec shell 命令 {} \;
    - find ./ -type d -exec ls -l {} \;
  - find ./ -type d -ok shell 命令 {} \;
    - find ./ -type d -ok ls -l {} \;
- 注意：{}中间不能又空格
- ok 比较安全，特别是在执行 rm 删除文件的时候
  - find ./ -type d | xargs shell 命令
    - find ./ -type d | xargs ls -l

## grep 命令

### grep -r(有目录) "查找的内容" 搜索路径

- -r 参数，若是目录，则可以递归搜索
- -n 参数可以显示该查找内容所在的行号
- -i 参数可以忽略大小写进行查找
- -v 参数不显示含有某字符串

### 搜索当前目录下包含 hello world 字符串的文件

- `grep -r -n "hello world" ./`    ----显示行号
- `grep -r -n -i "HELLO world" ./`    ----忽略大小写查找

## find 和 grep 命令结合使用

### 先使用 find 命令查找文件，然后使用 grep 命令查找那些文件包含某个字符串
- find . -name "*.c" | xargs grep -n "main"

## Linux 中常用的压缩工具

### gzip 和 bzip2

- 不能压缩目录，只能一个一个文件进行压缩，压缩之后会使源文件消失
- gzip * 压缩当前目录下所有的文件，但是目录不能压缩
- gunzip * 解压当前目录下所有的 .gz 文件
- bzip2 * 压缩当前目录下所有的文件，但是目录不能压缩
- bunzip2 * 解压当前目录下所有的 .bz2 文件

### tar 工具

- 相关参数说明
  - z：用 gzip 来压缩/解压文件
  - j：用 bzip2 来压缩/解压文件
  - c：create，创建新的压缩文件，与 x 互斥使用
  - x：从压缩文件中释放文件，与 c 互斥使用
  - f：指定压缩文件的名字
  - t：查看压缩包中有哪些文件
- 压缩：
  - tar -cvf 压缩包名.tar 要压缩的文件或目录
  - tar -cvzf 压缩包名.tar.gz 要压缩的文件或目录
  - tar -cvjf 压缩包名.tar.bz2 要压缩的文件或目录
- 解压缩
  - tar -xvf 压缩包名.tar
  - tar -xzvf 压缩包名.tar.gz
  - tar -xjvf 压缩包名.tar.bz2
  - 解压到指定目录：添加参数 -C(大写)
    - tar -xzvf 压缩包名.tar.gz -C 指定目录
- 查看压缩包中有哪些文件
  - tar -tvf 压缩包名.tar

### rar 工具

- 使用前需要安装 rar 工具
  - 安装命令：sudo apt-get install rar
- 压缩：
  - 命令： rar a -r 压缩包名 要压缩的文件
    - 压缩目录需要使用参数： -R
    - rar a -r my aa bb dir ----将 aa bb dir 三个文件压缩到 my.rar 中
  - 打包的生成的新文件不需要指定后缀
- 解压缩：
  - 命令：rar x xxx.rar 压缩目录
    - rar x my.rar ----将 my.rar 中的文件解压到当前目录
  - 解压到指定目录，直接指定解压目录即可
    - rar x xxx.rar 目录
    - rar x my.rar TAR ----将 my.rar 中的文件解压到 TAR 目录
    - 注意：若解压目录不存在则会报错

### zip 工具

- 压缩：zip -r 压缩包名 要压缩的文件或目录
  - 压缩目录需要使用参数 -R
  - 使用该命令不需要指定压缩包后缀
  - zip -r xxx file dir ----将 file dir 两个文件压缩到 xxx.zip 中
- 解压缩：unzip 压缩包名
  - 解压缩到指定目录：添加参数 -d 解压目录
  - unzip xxx.zip -d /home
  - 注意：若解压目录不存在则会创建

## 软件的安装和卸载

### 在线安装

- 软件安装：sudo apt-get install 软件名
- 软件卸载：sudo apt-get remove 软件名
- 更新软件列表：sudo apt-get update
- 清理安装包：sudo apt-get clean

### 安装包安装

- 在 Ubuntu 系统下必须有 deb 格式的安装包
- 软件安装：sudo dpkg -i xxx.deb
- 软件卸载：sudo dpkg -r xxx.deb