# 标准IO编程
本部分文件IO变成是基于流缓冲的（不基于文件描述符，带缓存），它是符合ANSI C 的标准I/O 处理。<br>

标准 I/O 提供流缓冲的目的是尽可能减少使用read()和write()等系统调用的数量。标准I/O 提供了3 种类型
的缓冲存储。

 全缓冲：在这种情况下，当填满标准 I/O 缓存后才进行实际I/O 操作。存放在磁盘上的文件通常
是由标准I/O 库实施全缓冲的。在一个流上执行第一次I/O 操作时，通常调用malloc()就是使用全
缓冲。<br>
 行缓冲：在这种情况下，当在输入和输出中遇到行结束符时，标准 I/O 库执行I/O 操作。这允许
我们一次输出一个字符（如fputc()函数），但只有写了一行之后才进行实际I/O 操作。标准输入和
标准输出就是使用行缓冲的典型例子。<br>
 不带缓冲：标准 I/O 库不对字符进行缓冲。如果用标准I/O 函数写若干字符到不带缓冲的流中，
则相当于用系统调用write()函数将这些字符全写到被打开的文件上。标准出错stderr 通常是不带
缓存的，这就使得出错信息可以尽快显示出来，而不管它们是否含有一个行结束符。

## 1．打开文件
打开文件有三个标准函数，分别为：fopen()、fdopen()和freopen()。它们可以以不同的模式打开，但都返
回一个指向FILE 的指针，该指针指向对应的I/O 流。

fopen()可以指定打开文件的路径和模式，fdopen()可以指定打开的文件描述符和模式，而freopen()
除可指定打开的文件、模式外，还可指定特定的I/O 流。

```c
所需头文件#include <stdio.h>
函数原型FILE * fopen(const char * path, const char * mode)
函数传入值
Path：包含要打开的文件路径及文件名
mode：文件打开状态
函数返回值
成功：指向 FILE 的指针
失败：NULL

mode:
```
r 或rb 打开只读文件，该文件必须存在
r＋或r＋b 打开可读写的文件，该文件必须存在
W 或wb 打开只写文件，若文件存在则文件长度清为0，即会擦写文件以前的内容。若文件不存在则建立该文件
w+或w＋b 打开可读写文件，若文件存在则文件长度清为0，即会擦写文件以前的内容。若文件不存在则建立该文件
a 或ab 以附加的方式打开只写文件。若文件不存在，则会建立该文件；如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留
a+或a＋b 以附加方式打开可读写的文件。若文件不存在，则会建立该文件；如果文件存在，写入的数据会被加到文件尾后，即文件原先的内容会被保留
```

加入 b 字符用来告诉函数库打开的文件为二进制文件，而非纯文本文件。不过在Linux系统中会自动识别不同类型的文件而将此符号忽略。

```c
所需头文件 #include <stdio.h>
函数原型FILE * fdopen(int fd, const char * mode)
函数传入值
fd：要打开的文件描述符
mode：文件打开状态（后面会具体说明）
函数返回值
成功：指向 FILE 的指针
失败：NULL
```

```c
所需头文件#include <stdio.h>
函数原型FILE * freopen(const char *path, const char * mode, FILE * stream)
函数传入值
path：包含要打开的文件路径及文件名
mode：文件打开状态（后面会具体说明）
stream：已打开的文件指针
函数返回值
成功：指向 FILE 的指针
失败：NULL
```

## 2．关闭文件
关闭标准流文件的函数为 fclose()，该函数将缓冲区内的数据全部写入到文件中，并释放系统所提供的文件资源。
```c
所需头文件#include <stdio.h>
函数原型int fclose(FILE * stream)
函数传入值 stream：已打开的文件指针
函数返回值
成功：0
失败：EOF
```
## 3．读文件
在文件流被打开之后，可对文件流进行读写等操作，其中读操作的函数为 fread()。
```c
所需头文件#include <stdio.h>
函数原型size_t fread(void * ptr,size_t size,size_t nmemb,FILE * stream)
函数传入值
ptr：存放读入记录的缓冲区
size：读取的记录大小
nmemb：读取的记录数
stream：要读取的文件流
函数返回值 成功：返回实际读取到的 nmemb 数目 失败：EOF
```

## 4．写文件
fwrite()函数用于对指定的文件流进行写操作。
```c
所需头文件#include <stdio.h>
函数原型size_t fwrite(const void * ptr,size_t size, size_t nmemb, FILE * stream)
函数传入值
ptr：存放写入记录的缓冲区
size：写入的记录大小
nmemb：写入的记录数
stream：要写入的文件流
函数返回值
成功：返回实际写入的记录数目
失败：EOF
```

## 5.汇总实例
```c
#include <stdlib.h>
#include <stdio.h>
#define BUFFER_SIZE 1024 /* 每次读写缓存大小 */
#define SRC_FILE_NAME "src_file" /* 源文件名 */
#define DEST_FILE_NAME "dest_file" /* 目标文件名文件名 */
#define OFFSET 10240 /* 复制的数据大小 */
int main()
{
    FILE *src_file, *dest_file;
    unsigned char buff[BUFFER_SIZE];
    int real_read_len;
/* 以只读方式打开源文件 */
    src_file = fopen(SRC_FILE_NAME, "r");
/* 以写方式打开目标文件，若此文件不存在则创建 */
    dest_file = fopen(DEST_FILE_NAME, "w");
    if (!src_file || !dest_file)
    {
        printf("Open file error\n");
        exit(1);
    }
/* 将源文件的读写指针移到最后10KB 的起始位置*/
    fseek(src_file, -OFFSET, SEEK_END);
/* 读取源文件的最后10KB 数据并写到目标文件中，每次读写1KB */
    while ((real_read_len = fread(buff, 1, sizeof(buff), src_file)) > 0)
    {
/*src_file 是读写指针，每次移动*/
        fwrite(buff, 1, real_read_len, dest_file);
    }
    fclose(dest_file);
    fclose(src_file);
    return 0;
}
```