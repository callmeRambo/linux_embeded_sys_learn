# Socket
在Linux 中的网络编程是通过socket 接口来进行的。socket 是一种特殊的I/O 接口，它也是一种文件描述符。socket 是一种常用的进程之间通信机制，通过它不仅能实现本地机器上的进程之间的通信，而且通过网络能够在不同机器上的进程之间进行通信。每一个 socket 都用一个半相关描述{协议、本地地址、本地端口}来表示；一个完整的套接字则用一个相关描述{协议、本地地址、本地端口、远程地址、远程端口}来表示。

## socket 类型
1. 流式socket（SOCK_STREAM）。
流式套接字提供可靠的、面向连接的通信流；它使用 TCP 协议，从而保证了数据传输的正确性和顺序性。
2. 数据报socket（SOCK_DGRAM）。
数据报套接字定义了一种无连接的服务，数据通过相互独立的报文进行传输，是无序的，并且不保证是可靠、无差错的。它使用数据报协议UDP。
3. 原始socket。
原始套接字允许对底层协议如 IP 或ICMP 进行直接访问，它功能强大但使用较为不便，主要用于一些协议的开发。

## 1.地址结构相关处理
两个重要的数据类型：sockaddr 和sockaddr_in，这两个结构类型都是用来保存socket 信息的，

```c
struct sockaddr
{
    unsigned short sa_family; /*地址族*/
    char sa_data[14]; /*14 字节的协议地址，包含该socket 的IP 地址和端口号。*/
};
struct sockaddr_in
{
    short int sa_family; /*地址族*/
    unsigned short int sin_port; /*端口号*/
    struct in_addr sin_addr; /*IP 地址*/
    unsigned char sin_zero[8]; /*填充0 以保持与struct sockaddr 同样大小*/
};
```
2. 结构字段。
```c
结构定义头文件#include <netinet/in.h>
sa_family
AF_INET：IPv4 协议
AF_INET6：IPv6 协议
AF_LOCAL：UNIX 域协议
AF_LINK：链路地址协议
AF_KEY：密钥套接字（socket）
```
## 2．数据存储优先顺序
计算机数据存储有两种字节优先顺序：高位字节优先（称为大端模式）和低位字节优先（称为小端模式，PC 机通常采用小端模式）。Internet 上数据以高位字节优先顺序在网络上传输，因此在有些情况下，需要对这两个字节存储优先顺序进行相互转化。这里用到了4 个函数：htons()、ntohs()、htonl()和ntohl()。这4个地址分别实现网络字节序和主机字节序的转化，这里的h 代表host，n 代表network，s 代表short，l 代表long。通常16 位的IP 端口号用s 代表，而IP 地址用l 来代表。

```c
所需头文件#include <netinet/in.h>
函数原型
uint16_t htons(unit16_t host16bit)
uint32_t htonl(unit32_t host32bit)
uint16_t ntohs(unit16_t net16bit)
uint32_t ntohs(unit32_t net32bit)
函数传入值 host16bit：主机字节序的16 位数据
host32bit：主机字节序的32 位数据
net16bit：网络字节序的16 位数据
net32bit：网络字节序的32 位数据
函数返回值
成功：返回要转换的字节序
出错：-1
```
## 3．地址格式转化
通常用户在表达地址时采用的是点分十进制表示的数值（或者是以冒号分开的十进制 IPv6 地址），而在通常使用的socket 编程中所使用的则是二进制值，这就需要将这两个数值进行转换。

这里在IPv4 中用到的函数有inet_aton()、inet_addr()和inet_ntoa()，而IPv4 和IPv6 兼容的函数有inet_pton()和inet_ntop()。
```c
所需头文件#include <arpa/inet.h>
函数原型int inet_pton(int family, const char *strptr, void *addrptr)
函数传入值
family
AF_INET：IPv4 协议
AF_INET6：IPv6 协议
strptr：要转化的值
addrptr：转化后的地址
函数返回值
成功：0
出错：-1

所需头文件#include <arpa/inet.h>
函数原型int inet_ntop(int family, void *addrptr, char *strptr, size_t len)
函数传入值family
AF_INET：IPv4 协议
AF_INET6：IPv6 协议
函数传入值
addrptr：转化后的地址
strptr：要转化的值
len：转化后值的大小
函数返回值
成功：0
出错：-1

```

## 4．名字地址转化
1. 函数说明

到IPv6 时，地址长度多达128 位，那时就更加不可能一次次记忆那么长的IP 地址了。因此，使用主机名将会是很好的选择。在Linux 中，同样有一些函数可以实现主机名和地址的转化，最为常见的有gethostbyname()、gethostbyaddr()和getaddrinfo()等，它们都可以实现IPv4 和IPv6 的地址和主机名之间的转化。其中gethostbyname()是将主机名转化为IP 地址，gethostbyaddr()则是逆操作，是将IP 地址转化为主机名，另外getaddrinfo()还能实现自动识别IPv4 地址和IPv6地址。

gethostbyname()和gethostbyaddr()都涉及一个hostent 的结构体.
```c
struct hostent
{
char *h_name;/*正式主机名*/
char **h_aliases;/*主机别名*/
int h_addrtype;/*地址类型*/
int h_length;/*地址字节长度*/
char **h_addr_list;/*指向IPv4 或IPv6 的地址指针数组*/
}
```
调用 gethostbyname()函数或gethostbyaddr()函数后就能返回hostent 结构体的相关信息。getaddrinfo()函数涉及一个addrinfo 的结构体
```c
struct addrinfo
{
int ai_flags;/*AI_PASSIVE, AI_CANONNAME;*/
int ai_family;/*地址族*/
int ai_socktype;/*socket 类型*/
int ai_protocol;/*协议类型*/
size_t ai_addrlen;/*地址字节长度*/
char *ai_canonname;/*主机名*/
struct sockaddr *ai_addr;/*socket 结构体*/
struct addrinfo *ai_next;/*下一个指针链表*/
}
```
2. 函数格式
```c
所需头文件#include <netdb.h>
函数原型struct hostent *gethostbyname(const char *hostname)
函数传入值 hostname：主机名
函数返回值
成功：hostent 类型指针
出错：-1

调用该函数时可以首先对 hostent 结构体中的h_addrtype 和h_length 进行设置，若为IPv4 可设置为AF_INET 和4；若为IPv6 可设置为AF_INET6 和16；若不设置则默认为IPv4 地址类型。
```
```c
所需头文件#include <netdb.h>
函数原型int getaddrinfo(const char *node, const char *service, const struct addrinfo *hints, struct addrinfo **result)
函数传入值
node：网络地址或者网络主机名
service：服务名或十进制的端口号字符串
hints：服务线索
result：返回结果
函数返回值
成功：0
出错：-1

addrinfo:
结构体头文件#include <netdb.h>
ai_flags
AI_PASSIVE：该套接口是用作被动地打开
AI_CANONNAME：通知getaddrinfo 函数返回主机的名字
ai_family
AF_INET：IPv4 协议
AF_INET6：IPv6 协议
AF_UNSPEC：IPv4 或IPv6 均可
ai_socktype SOCK_STREAM：字节流套接字socket（TCP）
SOCK_DGRAM：数据报套接字socket（UDP）
ai_protocol
IPPROTO_IP：IP 协议
IPPROTO_IPV4：IPv4 协议4 IPv4
IPPROTO_IPV6：IPv6 协议
IPPROTO_UDP：UDP
IPPROTO_TCP：TCP    
```
3. 使用实例
```c
/* getaddrinfo.c */
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
int main()
{
    struct addrinfo hints, *res = NULL;
    int rc;
    memset(&hints, 0, sizeof(hints));
/*设置addrinfo 结构体中各参数 */
    hints.ai_flags = AI_CANONNAME;
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_DGRAM;
    hints.ai_protocol = IPPROTO_UDP;
/*调用getaddinfo 函数*/
    rc = getaddrinfo("localhost", NULL, &hints, &res);
    if (rc != 0)
    {
        perror("getaddrinfo");
        exit(1);
    }
    else
    {
        printf("Host name is %s\n", res->ai_canonname);
    }
    exit(0);
}
```