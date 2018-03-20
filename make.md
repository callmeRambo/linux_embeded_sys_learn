# make 工程管理器
make 工程管理器也就是个“自动编译管理器”，这里的“自动”是指它能够根据文件时间戳自动
发现更新过的文件而减少编译的工作量，同时，它通过读入makefile 文件的内容来执行大量的编译工作。
## makefile 基本结构
makefile 是make 读入的惟一配置文件，因此本节的内容实际就是讲述makefile 的编写规则。在一个makefile
中通常包含如下内容：
```
 需要由 make 工具创建的目标体（target），通常是目标文件或可执行文件；
 要创建的目标体所依赖的文件（dependency_file）；
 创建每个目标体时需要运行的命令（command），这一行必须以制表符（tab 键）开头。
它的格式为：
target: dependency_files
    command /* 该行必须以tab 键开头*/
```
hello.o: hello.c hellp.h
    gcc -c hello.c -o hello.o
## makefile 变量
```
OBJS = kang.o yul.o
CC = gcc
CFLAGS = -Wall -O -g
david : $(OBJS)
    $(CC) $(OBJS) -o david
kang.o : kang.c kang.h
    $(CC) $(CFLAGS) -c kang.c -o kang.o
yul.o : yul.c yul.h
    $(CC) $(CFLAGS) -c yul.c -o yul.o
```
## makefile 规则
### 1．隐式规则
### 2．模式规则
```
OBJS = kang.o yul.o
CC = gcc
CFLAGS = -Wall -O -g
david : $(OBJS)
    $(CC) $^ -o $@
%.o : %.c
    (CC) $(CFLAGS) -c $< -o $@
```