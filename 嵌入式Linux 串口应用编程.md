# 串口应用编程
## 串口描述
常见的数据通信的基本方式可分为并行通信与串行通信两种。<br>
 并行通信是指利用多条数据传输线将一个字数据的各比特位同时传送。它的特点是传输速度快，
适用于传输距离短且传输速度较高的通信。<br>
 串行通信是指利用一条传输线将数据以比特位为单位顺序传送。特点是通信线路简单，利用简单
的线缆就可实现通信，降低成本，适用于传输距离长且传输速度较慢的通信。

在 Linux 中，所有的设备文件一般都位于“/dev”下，其中串口1 和串口2 对应的设备名依次为“/dev/ttyS0”
和“/dev/ttyS1”，而且USB 转串口的设备名通常为“/dev/ttyUSB0”和“/dev/ttyUSB1”（因版本不同该设
备名会有所不同），可以查看在“/dev”下的文件以确认。
## 串口设置
```c
＃include<termios.h>
struct termios
{
    unsigned short c_iflag; /* 输入模式标志 */
    unsigned short c_oflag; /* 输出模式标志 */
    unsigned short c_cflag; /* 控制模式标志*/
    unsigned short c_lflag; /* 本地模式标志 */
    unsigned char c_line; /* 线路规程 */
    unsigned char c_cc[NCC]; /* 控制特性 */
    speed_t c_ispeed; /* 输入速度 */
    speed_t c_ospeed; /* 输出速度 */
};
```
termios 是在POSIX 规范中定义的标准接口，表示终端设备（包括虚拟终端、串口等）。口是一种终端设备，
一般通过终端编程接口对其进行配置和控制。在具体讲解串口相关编程之前，先了解一下终端相关知识。
终端有 3 种工作模式，分别为规范模式（canonical mode）、非规范模式（non-canonical mode）和原始模式
（raw mode）。<br>

通过在 termios 结构的c_lflag 中设置ICANNON 标志来定义终端是以规范模式（设置ICANNON 标志）还是以非规范模式（清除ICANNON 标志）工作，默认情况为规范模式。<br>
在规范模式下，所有的输入是基于行进行处理。在用户输入一个行结束符（回车符、EOF 等）之前，系统调用
read()函数读不到用户输入的任何字符。如果在read()函数中被请求读取的数据字节数小于当前行可读取的字节数，则read()函数只会读取被请求的字节数，剩下的字节下次再被读取。

在非规范模式下，所有的输入是即时有效的，不需要用户另外输入行结束符，而且不可进行行编辑。在非规范模式下，对参数MIN（c_cc[VMIN]）和TIME（c_cc[VTIME]）的设置决定read()函数的调用方式。设置可以有4 种不同的情况。
```
 MIN = 0 和TIME = 0：read()函数立即返回。若有可读数据，则读取数据并返回被读取的字节数，
否则读取失败并返回0。
 MIN > 0 和TIME = 0：read()函数会被阻塞直到MIN 个字节数据可被读取。
 MIN = 0 和TIME > 0：只要有数据可读或者经过TIME 个十分之一秒的时间，read()函数则立即返
回，返回值为被读取的字节数。如果超时并且未读到数据，则read()函数返回0。
 MIN > 0 和TIME > 0：当有MIN 个字节可读或者两个输入字符之间的时间间隔超过TIME 个十
分之一秒时，read()函数才返回。因为在输入第一个字符之后系统才会启动定时器，所以在这种情
况下，read()函数至少读取一个字节之后才返回。
```

原始模式是一种特殊的非规范模式。在原始模式下，所有的输入数据以字节为单位被处理。即有一个字节输入时，触发输入有效。在这个模式下，终端是不可回显的，而且所有特定的终端输入/输出控制处理不可用。通过调用cfmakeraw()函数可以将终端设置为原始模式，而且该函数通过以下代码可以得到实现。
```c
termios_p->c_iflag &= ~(IGNBRK | BRKINT | PARMRK | ISTRIP
| INLCR | IGNCR | ICRNL | IXON);
termios_p->c_oflag &= ~OPOST;
termios_p->c_lflag &= ~(ECHO | ECHONL | ICANON | ISIG | IEXTEN);
termios_p->c_cflag &= ~(CSIZE | PARENB);
termios_p->c_cflag |= CS8;
```

波特率:单片机或计算机在串口通信时的速率。指的是信号被调制以后在单位时间内的变化，即单位时间内载波参数变化的次数，如每秒钟传送240个字符，而每个字符格式包含10位（1个起始位，1个停止位，8个数据位），这时的波特率为240Bd，比特率为10位*240个/秒=2400bps。

下面讲解设置串口的基本方法。设置串口中最基本的包括波特率设置，校验位和停止位设置。在这个结构中最为重要的是c_cflag，通过对它的赋值，用户可以设置波特率、字符大小、数据位、停止位、奇偶校验位和硬软流控等。另外c_iflag 和c_cc 也是比较常用的标志。在此主要对这3 个成员进行详细说明。
```
c_cflag支持的常量名称
CBAUD 波特率的位掩码
B0 0 波特率（放弃DTR）
… …
B1800 1800 波特率
B2400 2400 波特率
B4800 4800 波特率
B9600 9600 波特率
B19200 19200 波特率
B38400 38400 波特率
B57600 57600 波特率
B115200 115200 波特率
EXTA 外部时钟率
EXTB 外部时钟率
CSIZE 数据位的位掩码
CS5 5 个数据位
CS6 6 个数据位
CS7 7 个数据位
CS8 8 个数据位
CSTOPB 2 个停止位（不设则是1 个停止位）
CREAD 接收使能
PARENB
PARODD
校验位使能
使用奇校验而不使用偶校验
HUPCL 最后关闭时挂线（放弃DTR）
CLOCAL 本地连接（不改变端口所有者）
CRTSCTS 硬件流控
```

不能直接对 c_cflag 成员初始化，而要将其通过“与”、“或”操作使用其中的某些选项。

输入模式标志c_iflag 用于控制端口接收端的字符输入处理。
```
c_iflag支持的常量名称
INPCK 奇偶校验使能
IGNPAR 忽略奇偶校验错误
PARMRK 奇偶校验错误掩码
ISTRIP 裁减掉第8 位比特
IXON 启动输出软件流控
IXOFF 启动输入软件流控
IXANY 输入任意字符可以重新启动输出（默认为输入起始字符才重启输出）
IGNBRK 忽略输入终止条件
BRKINT 当检测到输入终止条件时发送SIGINT 信号
INLCR 将接收到的NL（换行符）转换为CR（回车符）
IGNCR 忽略接收到的CR（回车符）
ICRNL 将接收到的CR（回车符）转换为NL（换行符）
IUCLC 将接收到的大写字符映射为小写字符
IMAXBEL 当输入队列满时响铃
```

c_oflag 用于控制终端端口发送出去的字符处理
```
c_oflag 支持的常量名称
OPOST 启用输出处理功能，如果不设置该标志，则其他标志都被忽略
OLCUC 将输出中的大写字符转换成小写字符
ONLCR 将输出中的换行符（‘\n’）转换成回车符（‘\r’）
ONOCR 如果当前列号为0，则不输出回车符
OCRNL 将输出中的回车符（‘\r’）转换成换行符（‘\n’）
ONLRET 不输出回车符
OFILL 发送填充字符以提供延时
OFDEL 如果设置该标志，则表示填充字符为DEL 字符，否则为NUL 字符
NLDLY 换行延时掩码
CRDLY 回车延时掩码
TABDLY 制表符延时掩码
BSDLY 水平退格符延时掩码
VTDLY 垂直退格符延时掩码
FFLDY 换页符延时掩码
```

c_lflag 用于控制控制终端的本地数据处理和工作模式
```
c_lflag支持的常量名称
ISIG 若收到信号字符（INTR、QUIT 等），则会产生相应的信号
ICANON 启用规范模式
ECHO 启用本地回显功能
ECHOE 若设置ICANON，则允许退格操作
ECHOK 若设置ICANON，则KILL 字符会删除当前行
ECHONL 若设置ICANON，则允许回显换行符
ECHOCTL
若设置ECHO，则控制字符（制表符、换行符等）会显示成“^X”，其中X 的ASCII
码等于给相应控制字符的ASCII 码加上0x40。例如：退格字符（0x08）会显示为
“^H”（’H’的ASCII 码为0x48）
ECHOPRT 若设置ICANON 和IECHO，则删除字符（退格符等）和被删除的字符都会被显示
ECHOKE 若设置ICANON，则允许回显在ECHOE 和ECHOPRT 中设定的KILL 字符
NOFLSH 在通常情况下，当接收到INTR、QUIT 和SUSP 控制字符时，会清空输入和输出
队列。如果设置该标志，则所有的队列不会被清空
TOSTOP 若一个后台进程试图向它的控制终端进行写操作，则系统向该后台进程的进程组
发送SIGTTOU 信号。该信号通常终止进程的执行
IEXTEN 启用输入处理功能
```

c_cc 定义特殊控制特性
```
c_cc支持的常量名称
VINTR 中断控制字符，对应键为CTRL+C
VQUIT 退出操作符，对应键为CRTL+Z
VERASE 删除操作符，对应键为Backspace（BS）
VKILL 删除行符，对应键为CTRL+U
VEOF 文件结尾符，对应键为CTRL+D
VEOL 附加行结尾符，对应键为Carriage return（CR）
VEOL2 第二行结尾符，对应键为Line feed（LF）
VMIN 指定最少读取的字符数
VTIME 指定读取的每个字符之间的超时时间
```

# 设置串口属性的基本流程
## 1．保存原先串口配置
首先，为了安全起见和以后调试程序方便，可以先保存原先串口的配置，在这里可以使用函数tcgetattr(fd,
&old_cfg)。该函数得到fd 指向的终端的配置参数，并将它们保存于termios 结构变量old_cfg 中。该函数
还可以测试配置是否正确、该串口是否可用等。若调用成功，函数返回值为0，若调用失败，函数返回值
为-1.
```c
if (tcgetattr(fd, &old_cfg) != 0)
{
    perror("tcgetattr");
    return -1;
}
```
## 2．激活选项
CLOCAL 和CREAD 分别用于本地连接和接受使能，因此，首先要通过位掩码的方式激活这两个选项。
```
newtio.c_cflag |= CLOCAL | CREAD;
调用 cfmakeraw()函数可以将终端设置为原始模式，在后面的实例中，采用原始模式进行串口数据通信。
cfmakeraw(&new_cfg);
```
## 3．设置波特率
设置波特率有专门的函数，用户不能直接通过位掩码来操作。设置波特率的主要函数有：cfsetispeed()和
cfsetospeed()。
```c
cfsetispeed(&new_cfg, B115200);
cfsetospeed(&new_cfg, B115200);
```
一般地，用户需将终端的输入和输出波特率设置成一样的。这几个函数在成功时返回0，失败时返回1。
## 4．设置字符大小
与设置波特率不同，设置字符大小并没有现成可用的函数，需要用位掩码。一般首先去除数据位中的位掩
码，再重新按要求设置。如下所示：
```c
new_cfg.c_cflag &= ~CSIZE; /* 用数据位掩码清空数据位设置 */
new_cfg.c_cflag |= CS8;
```
## 5．设置奇偶校验位
设置奇偶校验位需要用到termios 中的两个成员：c_cflag 和c_iflag。首先要激活c_cflag 中的校验位使能标
志PARENB 和是否要进行偶校验，同时还要激活c_iflag 中的对于输入数据的奇偶校验使能（INPCK）
```c
new_cfg.c_cflag |= (PARODD | PARENB);
new_cfg.c_iflag |= INPCK;
而使能偶校验时，代码如下所示：
new_cfg.c_cflag |= PARENB;
new_cfg.c_cflag &= ~PARODD; /* 清除偶校验标志，则配置为奇校验*/
new_cfg.c_iflag |= INPCK;
```
## 6．设置停止位
设置停止位是通过激活c_cflag 中的CSTOPB 而实现的。若停止位为一个，则清除CSTOPB，若停止位为两
个，则激活CSTOPB。
```c
new_cfg.c_cflag &= ~CSTOPB; /* 将停止位设置为一个比特 */
new_cfg.c_cflag |= CSTOPB; /* 将停止位设置为两个比特 */
```
## 7．设置最少字符和等待时间
在对接收字符和等待时间没有特别要求的情况下，可以将其设置为0，则在任何情况下read()函数立即返回，
如下所示：
```c
new_cfg.c_cc[VTIME] = 0;
new_cfg.c_cc[VMIN] = 0;
```
## 8．清除串口缓冲
由于串口在重新设置之后，需要对当前的串口设备进行适当的处理，这时就可调用在<termios.h>中声明的
tcdrain()、tcflow()、tcflush()等函数来处理目前串口缓冲中的数据，它们的格式如下所示。
```c
int tcdrain(int fd); /* 使程序阻塞，直到输出缓冲区的数据全部发送完毕*/
int tcflow(int fd, int action) ; /* 用于暂停或重新开始输出 */
int tcflush(int fd, int queue_selector); /* 用于清空输入/输出缓冲区*/
```
在本实例中使用 tcflush()函数，对于在缓冲区中的尚未传输的数据，或者收到的但是尚未读取的数据，其
处理方法取决于queue_selector 的值，它可能的取值有以下几种。
```
 TCIFLUSH：对接收到而未被读取的数据进行清空处理。
 TCOFLUSH：对尚未传送成功的输出数据进行清空处理。
 TCIOFLUSH：包括前两种功能，即对尚未处理的输入输出数据进行清空处理。
```
## 9．激活配置
在完成全部串口配置之后，要激活刚才的配置并使配置生效。这里用到的函数是tcsetattr()，它的函数原型
是：
```c
tcsetattr(int fd, int optional_actions, const struct termios *termios_p);
```
其中参数 termios_p 是termios 类型的新配置变量。
```
参数 optional_actions 可能的取值有以下3 种：
 TCSANOW：配置的修改立即生效。
 TCSADRAIN：配置的修改在所有写入fd 的输出都传输完毕之后生效。
 TCSAFLUSH：所有已接受但未读入的输入都将在修改生效之前被丢弃。
```
该函数若调用成功则返回 0，若失败则返回1，代码如下所示：
```c
if ((tcsetattr(fd, TCSANOW, &new_cfg)) != 0)
{
    perror("tcsetattr");
    return -1;
}
```
## 设置函数汇总：
下面给出了串口配置的完整函数。通常，为了函数的通用性，通常将常用的选项都在函数中列出，这样可
以大大方便以后用户的调试使用。该设置函数如下所示：
```c
int set_com_config(int fd,int baud_rate,
int data_bits, char parity, int stop_bits)
{
    struct termios new_cfg,old_cfg;
    int speed;
/*保存并测试现有串口参数设置，在这里如果串口号等出错，会有相关的出错信息*/
    if (tcgetattr(fd, &old_cfg) != 0)
    {
        perror("tcgetattr");
        return -1;
    }
/* 设置字符大小*/
    new_cfg = old_cfg;
    cfmakeraw(&new_cfg); /* 配置为原始模式 */
    new_cfg.c_cflag &= ~CSIZE;
/*设置波特率*/
    switch (baud_rate)
    {
        case 2400:
        {
            speed = B2400;
        }
        break;
        case 4800:
        {
            speed = B4800;
        }
        break;
        case 9600:
        {
            speed = B9600;
        }
        break;
        case 19200:
        {
            speed = B19200;
        }
        break;
        case 38400:
        {
            speed = B38400;
        }
        break;
        default:
        case 115200:
        {
            speed = B115200;
        }
        break;
        }
    cfsetispeed(&new_cfg, speed);
    cfsetospeed(&new_cfg, speed);
        /*设置停止位*/
    switch (data_bits)
    {
        case 7:
        {
            new_cfg.c_cflag |= CS7;
        }
        break;
        default:
        case 8:
        {
            new_cfg.c_cflag |= CS8;
        }
        break;
    }
            /*设置奇偶校验位*/
    switch (parity)
    {
        default:
        case 'n':
        case 'N':
        {
            new_cfg.c_cflag &= ~PARENB;
            new_cfg.c_iflag &= ~INPCK;
        }
        break;
        case 'o':
        case 'O':
        {
            new_cfg.c_cflag |= (PARODD | PARENB);
            new_cfg.c_iflag |= INPCK;
        }
        break;
        case 'e':
        case 'E':
        {
            new_cfg.c_cflag |= PARENB;
            new_cfg.c_cflag &= ~PARODD;
            new_cfg.c_iflag |= INPCK;
        }
        break;
        case 's': /*as no parity*/
        case 'S':
        {
            new_cfg.c_cflag &= ~PARENB;
            new_cfg.c_cflag &= ~CSTOPB;
        }
        break;
    }
        /*设置停止位*/
    switch (stop_bits)
    {
        default:
        case 1:
        {
            new_cfg.c_cflag &= ~CSTOPB;
        }
        break;
        case 2:
        {
            new_cfg.c_cflag |= CSTOPB;
        }
    }
            /*设置等待时间和最小接收字符*/
    new_cfg.c_cc[VTIME] = 0;
    new_cfg.c_cc[VMIN] = 1;
/*处理未接收字符*/
    tcflush(fd, TCIFLUSH);
/*激活新配置*/
    if ((tcsetattr(fd, TCSANOW, &new_cfg)) != 0)
    {
        perror("tcsetattr");
        return -1;
    }
    return 0;
}
```
# 串口使用
在配置完串口的相关属性后，就可以对串口进行打开和读写操作了。它所使用的函数和普通文件的读写函
数一样，都是open()、write()和read()。它们之间的区别的只是串口是一个终端设备，因此在选择函数的具
体参数时会有一些区别。另外，这里会用到一些附加的函数，用于测试终端设备的连接情况等。

## 1．打开串口
```c
fd = open( "/dev/ttyS0", O_RDWR|O_NOCTTY|O_NDELAY);
可以看到，这里除了普通的读写参数外，还有两个参数 O_NOCTTY 和O_NDELAY。

 O_NOCTTY 标志用于通知Linux 系统，该参数不会使打开的文件成为这个进程的控制终端。如果没有指定这个标志，那么任何一个输入（诸如键盘中止信号等）都将会影响用户的进程。
 O_NDELAY 标志通知Linux 系统，这个程序不关心DCD 信号线所处的状态（端口的另一端是否激活或者停止）。如果用户指定了这个标志，则进程将会一直处在睡眠状态，直到DCD 信号线被激活。

DCD也叫接收线信号检出(Received Line detection-RLSD)。当本地的DCE收到由通信链路另一端的DCE送来的载波信号时，使DCD变为有效(高电平)，表示已检测出远端的载波信号，要求数据终端设备(DTE)准备接收。

DCE 英文全称 Data Circuit-terminating Equipment，数字通信设备，通常指调制解调器，多路复用器或数字设备 设备，DCE为DTE提供时钟，借此DTE才能工作，比如 PC DCE 一方提供时钟 提供时钟 机和 MODEM 之间的连接。PC 机就是一个 DTE，MODEM 是一个 DCE DCE。
```
接下来可恢复串口的状态为阻塞状态，用于等待串口数据的读入，可用 fcntl()函数实现，如下所示：
```c
fcntl(fd, F_SETFL, 0);
```2
再接着可以测试打开文件描述符是否连接到一个终端设备，以进一步确认串口是否正确打开
```c
isatty(STDIN_FILENO);
```
该函数调用成功则返回 0，若失败则返回-1。<br>
这时，一个串口就已经成功打开了。接下来就可以对这个串口进行读和写操作。
```c
/*打开串口函数*/
int open_port(int com_port)
{
int fd;
#if (COM_TYPE == GNR_COM) /* 使用普通串口 */
    char *dev[] = {"/dev/ttyS0", "/dev/ttyS1", "/dev/ttyS2"};
```

打开串口的程序汇总：
```c
/*打开串口函数*/
int open_port(int com_port)
{
    int fd;
    #if (COM_TYPE == GNR_COM) /* 使用普通串口 */
    char *dev[] = {"/dev/ttyS0", "/dev/ttyS1", "/dev/ttyS2"};
    #else /* 使用USB 转串口 */
    char *dev[] = {"/dev/ttyUSB0", "/dev/ttyUSB1", "/dev/ttyUSB2"};
    #endif
    if ((com_port < 0) || (com_port > MAX_COM_NUM))
    {
        return -1;
    }
/* 打开串口 */
    fd = open(dev[com_port - 1], O_RDWR|O_NOCTTY|O_NDELAY);
    if (fd < 0)
    {
        perror("open serial port");
        return(-1);
    }
/*恢复串口为阻塞状态*/
    if (fcntl(fd, F_SETFL, 0) < 0)
    {
        perror("fcntl F_SETFL\n");
    }
/*测试是否为终端设备*/
    if (isatty(STDIN_FILENO) == 0)
    {
        perror("standard input is not a terminal device");
    }
    return fd;
}
```
## 2．读写串口
读写串口操作和读写普通文件一样，使用read()和write()函数即可，如下所示：
```c
write(fd, buff, strlen(buff));
read(fd, buff, BUFFER_SIZE);
```
```c
/* com_writer.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>
#include "uart_api.h"
int main(void)
{
    int fd;
    char buff[BUFFER_SIZE];
    if((fd = open_port(HOST_COM_PORT)) < 0) /* 打开串口 */
    {
        perror("open_port");
        return 1;
    }
    if(set_com_config(fd, 115200, 8, 'N', 1) < 0) /* 配置串口 */
    {
        perror("set_com_config");
        return 1;
    }
    do
    {
        printf("Input some words(enter 'quit' to exit):");
        memset(buff, 0, BUFFER_SIZE);
        if (fgets(buff, BUFFER_SIZE, stdin) == NULL)
        {
            perror("fgets");
            break;
        }
        write(fd, buff, strlen(buff));
    } while(strncmp(buff, "quit", 4));
    close(fd);
    return 0;
}
```
```c
/* com_reader.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>
#include "uart_api.h"
int main(void)
{
    int fd;
    char buff[BUFFER_SIZE];
    if((fd = open_port(TARGET_COM_PORT)) < 0) /* 打开串口 */
    {   
        perror("open_port");
        return 1;
    }
    if(set_com_config(fd, 115200, 8, 'N', 1) < 0) /* 配置串口 */
    {
        perror("set_com_config");
        return 1;
    }
    do
    {
        memset(buff, 0, BUFFER_SIZE);
        if (read(fd, buff, BUFFER_SIZE) > 0)
        {
            printf("The received words are : %s", buff);
        }
    } while(strncmp(buff, "quit", 4));
    close(fd);
    return 0;
}
```

```c
/* 宿主机，写串口*/
$ ./com_writer
Input some words(enter 'quit' to exit):hello, Reader!
Input some words(enter 'quit' to exit):I'm Writer!
Input some words(enter 'quit' to exit):This is a serial port testing program.
Input some words(enter 'quit' to exit):quit
/* 目标板，读串口*/
$ ./com_reader
The received words are : hello, Reader!
The received words are : I'm Writer!
The received words are : This is a serial port testing program.
The received words are : quit
```