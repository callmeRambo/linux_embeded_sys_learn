# 文件 I/O 编程

## Linux 系统调用及用户编程接口（API）
### 系统调用
系统调用是指操作系统提供给用户程序调用的一组“特殊”接口，用户程序可以通过这组“特殊”接口来获得操作系统内核提供的服务。例如用户可以通过进程控制相关的系统调用来创建进程、实现进程调度、进程管理等。

为了更好地保护内核空间，将程序的运行空间分为内核空间和用户空间（也就是常称的内核态和用户态），它们分别运行在不同的级别上，在逻辑上是相互隔离的。因此，用户进程在通常情况下不允许访问内核数据，也无法使用内核函数，它们只能在用户空间操作用户数据，调用用户空间的函数。

但是，在有些情况下，用户空间的进程需要获得一定的系统服务（调用内核空间程序），这时操作系统就必须利用系统提供给用户的“特殊接口”——系统调用规定用户进程进入内核空间的具体位置。进行系统调用时，程序运行空间需要从用户空间进入内核空间，处理完后再返回用户空间。

系统调用按照功能逻辑大致可分为进程控制、进程间通信、文件系统控制、系统控制、存
储管理、网络管理、socket 控制、用户管理等几类。

### Linux 中文件及文件描述符概述
Linux 中的文件主要分为4 种：普通文件、目录文件、链接文件和设备文件。

文件描述符。对于Linux 而言，所有对设备和文件的操作都是使用文件描述符来进行的。文件描述符是一个非负的整数，它是一个索引值，并指向在内核中每个进程打开文件的记录表。

当打开一个现存文件或创建一个新文件时，内核就向进程返回一个文件描述符；当需要读写文件时，也需要把文件描述符作为参数传递给相应的函数。

## 底层文件I/O 操作
open()、read()、write()、lseek()和close()。这
些函数的特点是不带缓存，直接对文件（包括设备）进行读写操作。这些函数虽然不是ANSI C 的组成部分，但是是POSIX 的组成部分。

## 基本文件操作
```c
#include <sys/types.h> /* 提供类型pid_t 的定义 */
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *pathname, int flags, int perms)
pathname 被打开的文件名（可包括路径名）

flag：文件打开的方式:
O_RDONLY：以只读方式打开文件
O_WRONLY：以只写方式打开文件
O_RDWR：以读写方式打开文件
O_CREAT：如果该文件不存在，就创建一个新的文件，并用第三个参数为其设置权限
O_EXCL：如果使用O_CREAT 时文件存在，则可返回错误消息。这一参数可测试文件是否存在。此时open 是原子操作，防止多个进程同时创建同一个文件
O_NOCTTY：使用本参数时，若文件为终端，那么该终端不会成为调用open()的那个进程的控制终端
O_TRUNC：若文件已经存在，那么会删除文件中的全部原有数据，并且设置文件大小为0。
O_APPEND：以添加方式打开文件，在打开文件的同时，文件指针指向文件的末尾，即将写入的数据添加到文件的末尾

perms被打开文件的存取权限:
可以用一组宏定义：S_I(R/W/X)(USR/GRP/OTH)
其中R/W/X 分别表示读/写/执行权限
USR/GRP/OTH 分别表示文件所有者/文件所属组/其他用户
例如，S_IRUSR | S_IWUSR 表示设置文件所有者的可读可写属
性。八进制表示法中600 也表示同样的权限

成功：返回文件描述符
失败：-1
```
```c
#include <unistd.h>
int close(int fd)
```
```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count)
buf：指定存储器读出数据的缓冲区
count：指定读出的字节数

成功：读到的字节数
0：已到达文件尾
-1：出错
在读普通文件时，若读到要求的字节数之前已到达文件的尾部，则返回的字节数会小于希望读出的字节数。

```

```c
#include <unistd.h>
ssize_t write(int fd, void *buf, size_t count)
buf：指定存储器写入数据的缓冲区
```

```c
#include <unistd.h>
#include <sys/types.h>
off_t lseek(int fd, off_t offset, int whence)
offset：偏移量，每一读写操作所需要移动的距离，单位是字节，可正可负（向前移，向后移）

whence：当前位置的基点
SEEK_SET：当前位置为文件的开头，新位置为偏移量的大小
SEEK_CUR：当前位置为文件指针的位置，新位置为当前位置加上
偏移量
SEEK_END：当前位置为文件的结尾，新位置为文件的大小加上偏
移量的大小

成功：文件的当前位移
-1：出错
```
读取文件制定字节并写入其他文件：
```c
/* copy_file.c */
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#define BUFFER_SIZE 1024 /* 每次读写缓存大小，影响运行效率*/
#define SRC_FILE_NAME "src_file" /* 源文件名 */
#define DEST_FILE_NAME "dest_file" /* 目标文件名文件名 */
#define OFFSE 10240 /* 复制的数据大小 */
int main()
{
    int src_file, dest_file;
    unsigned char buff[BUFFER_SIZE];
    int real_read_len;
/* 以只读方式打开源文件 */
    src_file = open(SRC_FILE_NAME, O_RDONLY);
/* 以只写方式打开目标文件，若此文件不存在则创建该文件, 访问权限值为644 */
    dest_file = open(DEST_FILE_NAME,
    O_WRONLY|O_CREAT, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH);
    if (src_file < 0 || dest_file < 0)
    {
        printf("Open file error\n");
        exit(1);
    }
/* 将源文件的读写指针移到最后10KB 的起始位置*/
    lseek(src_file, -OFFSET, SEEK_END);
/* 读取源文件的最后10KB 数据并写到目标文件中，每次读写1KB */
    while ((real_read_len = read(src_file, buff, sizeof(buff))) > 0)
    {
        write(dest_file, buff, real_read_len);
    }
    close(dest_file);
    close(src_file);
    return 0;
}
```

## 文件锁
在文件已经共享的情况下如何操作，也就是当多个用户共同使用、操作一个文件的情况.
fcntl() ：给文件上锁，来避免共享的资源产生竞争的状态。

文件锁包括建议性锁和强制性锁。建议性锁要求每个上锁文件的进程都要检查是否有锁存在，并尊重现有锁。在一般情况下，内核和系统都不使用建议性锁。强制性锁是由内核执行的锁，
当一个文件被上锁进行写入操作的时候，内核将阻止其他任何文件对其进行读写操作。采用强制性锁对性能的影响很大，每次读写操作都必须检查是否有锁存在。

在 Linux 中，实现文件上锁的函数有lockf()和fcntl()，其中lockf()用于对文件施加建议性锁，而fcntl()不仅可以施加建议性锁，还可以施加强制锁。同时，fcntl()还能对文件的某一记录上锁，也就是记录锁。

记录锁又可分为读取锁和写入锁。

读取锁又称为共享锁，它能够使多个进程都能在文件的同一部分建立读取锁。
写入锁又称为排斥锁，在任何时刻只能有一个进程在文件的某个部分上建立写入锁。
在文件的同一部分不能同时建立读取锁和写入锁。

```c
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
int fcnt1(int fd, int cmd, struct flock *lock)

cmd:
F_DUPFD：复制文件描述符
F_GETFD：获得fd 的close-on-exec 标志，若标志未设置，则文件经过exec()函
数之后仍保持打开状态
F_SETFD：设置close-on-exec 标志，该标志由参数arg 的FD_CLOEXEC 位决定
F_GETFL：得到open 设置的标志
F_SETFL：改变open 设置的标志
F_GETLK：根据lock 参数值，决定是否上文件锁
F_SETLK：设置lock 参数值的文件锁
F_SETLKW：这是F_SETLK 的阻塞版本（命令名中的W 表示等待（wait））。
在无法获取锁时，会进入睡眠状态；如果可以获取锁或者捕捉到信号则会返回

lock：结构为flock，设置记录锁的具体状态
struct flock
{
    short l_type;
    off_t l_start;
    short l_whence;
    off_t l_len;
    pid_t l_pid;
}
l_type:
F_RDLCK：读取锁（共享锁）
F_WRLCK：写入锁（排斥锁）
F_UNLCK：解锁

l_stat 相对位移量（字节）

l_whence：相对位移量的起点（同lseek的whence）
SEEK_SET：当前位置为文件的开头，新位置为偏移量的大小
SEEK_CUR：当前位置为文件指针的位置，新位置为当前位置加上偏移量
SEEK_END：当前位置为文件的结尾，新位置为文件的大小加上偏移量的大小

l_len 加锁区域的长度

0：成功
-1：出错
```

fcntl()使用实例:
下面首先给出了使用 fcntl()函数的文件记录锁功能的代码实现。在该代码中，首先给flock 结构体的对应位赋予相应的值。<br>
接着使用两次fcntl()函数，分别用于判断文件是否可以上锁和给相关文件上锁，这里用到的cmd 值分别为F_GETLK 和F_SETLK（或F_SETLKW）。<br>
用 F_GETLK 命令判断是否可以进行flock 结构所描述的锁操作：若可以进行，则flock 结构的l_type 会被设置为F_UNLCK，其他域不变；若不可行，则l_pid 被设置为拥有文件锁的进程号，其他域不变。<br>
用 F_SETLK 和F_SETLKW 命令设置flock 结构所描述的锁操作，后者是前者的阻塞版。
```c
/* lock_set.c */
int lock_set(int fd, int type)
{
    struct flock old_lock, lock;
    lock.l_whence = SEEK_SET;
    lock.l_start = 0;
    lock.l_len = 0;
    lock.l_type = type;
    lock.l_pid = -1;
/* 判断文件是否可以上锁 */
    fcntl(fd, F_GETLK, &lock);
    if (lock.l_type != F_UNLCK)
    {
/* 判断文件不能上锁的原因 */
        if (lock.l_type == F_RDLCK) /* 该文件已有读取锁 */
        {
            printf("Read lock already set by %d\n", lock.l_pid);
        }
        else if (lock.l_type == F_WRLCK) /* 该文件已有写入锁 */
        {
            printf("Write lock already set by %d\n", lock.l_pid);
        }
    }
/* l_type 可能已被F_GETLK 修改过 */
    lock.l_type = type;
/* 根据不同的type 值进行阻塞式上锁或解锁 */
    if ((fcntl(fd, F_SETLKW, &lock)) < 0)
    {
        printf("Lock failed:type = %d\n", lock.l_type);
        return 1;
    }
    
    switch(lock.l_type)
    {
    case F_RDLCK:
    {
        printf("Read lock set by %d\n", getpid());
    }
    break;

    case F_WRLCK:
    {
        printf("Write lock set by %d\n", getpid());
    }
    break;

    case F_UNLCK:
    {
        printf("Release lock by %d\n", getpid());
        return 1;
    }
    break;
    
    default:
    break;
    }/* end of switch */
    return 0;
}
```

```c
/* write_lock.c */
#include <unistd.h>
#include <sys/file.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include "lock_set.c"
int main(void)
{
    int fd;
/* 首先打开文件 */
    fd = open("hello",O_RDWR | O_CREAT, 0644);
    if(fd < 0)
    {
        printf("Open file error\n");
        exit(1);
    }
/* 给文件上写入锁 */
    lock_set(fd, F_WRLCK);
    getchar();
/* 给文件解锁 */
    lock_set(fd, F_UNLCK);
    getchar();
    close(fd);
    exit(0);
}
```

```c
/* fcntl_read.c */
#include <unistd.h>
#include <sys/file.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include "lock_set.c"
int main(void)
{
    int fd;
    fd = open("hello",O_RDWR | O_CREAT, 0644);
    if(fd < 0)
    {
        printf("Open file error\n");
        exit(1);
    }
/* 给文件上读取锁 */
    lock_set(fd, F_RDLCK);
    getchar();
/* 给文件解锁 */
    lock_set(fd, F_UNLCK);
    getchar();
    close(fd);
    exit(0);
}
```