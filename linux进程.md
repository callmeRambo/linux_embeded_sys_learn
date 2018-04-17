# linux 进程
## 1. 基本概念
1. 
进程是一个程序的一次执行的过程,同时也是
资源分配的最小单元。

是通过进程控制块来描述的。进程控制块包含了进程的描述信息、控制信息以及资源信息,它是进程
的一个静态描述。

2.
在 Linux 中最主要的进程标识有进程号(PID, Process Idenity Number)和它的父进程号(PPID, parent process
ID)
。其中 PID 惟一地标识一个进程。PID 和 PPID 都是非零的正整数。

pid and ppid c program:
```c
/* pid.c */
#include<stdio.h>
#include<unistd.h>
#include <stdlib.h>
int main()
{
/*获得当前进程的进程 ID 和其父进程 ID*/
printf("The PID of this process is %d\n", getpid());
printf("The PPID of this process is %d\n", getppid());
}
```
3.
进程运行的状态:
进程是程序的执行过程,根据它的生命周期可以划分成 3 种状态。

 执行态:该进程正在运行,即进程正在占用 CPU。

 就绪态:进程已经具备执行的一切条件,正在等待分配 CPU 的处理时间片。

 等待态:进程不能使用 CPU,若等待事件发生(等待的资源分配到)则可将其唤醒。

4. 进程结构
Linux 中的进程包含 3 个段,分别为“数据段”、
“代码段”和“堆栈段”

 “数据段”存放的是全局变量、常数以及动态数据
分配的数堆据空间,根据存放的数据,数据段又可以分成普通数据段(包BSS数据段括可读可写/只读数据段,存放静态初始化的全局变量或常量)、BSS 数据段(存放未初始化的全局变量)以及堆(存放数据段(可读/只读)动态分配的数据)。

 “代码段”存放的是程序代码的数据。

 “堆栈段”存放的是子程序的返回地址、子程序的参 低地址数以及程代码段序的局部变量等。

5. 进程的模式和类型
在 Linux 系统中,进程的执行模式划分为用户模式和内核模式。如果当前运行的是用户程序、应用程序或者
内核之外的系统程序,那么对应进程就在用户模式下运行;如果在用户程序执行过程中出现系统调用或者发
生中断事件,那么就要运行操作系统(即核心)程序,进程模式就变成内核模式。在内核模式下运行的进程
可以执行机器的特权指令,而且此时该进程的运行不受用户的干扰,即使是 root 用户也不能干扰内核模式下
进程的运行。
用户进程既可以在用户模式下运行,也可以在内核模式下运行

## 2.进程管理
1. 启动进程	: 手工, 调度
2. 调度进程:ps, top,nice, renice, kill, crontab, bg
3. Linux 进程控制编程

fork(): 使用 fork()
函数得到的子进程是父进程的一个复制品,它从父进程处继承了整个进程的地址空间,包括进程上下文、
代码段、进程堆栈、内存信息、打开的文件描述符、信号控制设定、进程优先级、进程组号、当前工作目
录、根目录、资源限制和控制终端等,而子进程所独有的只有它的进程号、资源使用和计时器等。
```c
/* fork.c */
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    pid_t result;
/*调用 fork()函数*/
    result = fork();
/*通过 result 的值来判断 fork()函数的返回情况,首先进行出错处理*/
    if(result == -1)
    {
        printf("Fork error\n");
    }
    else if (result == 0) /*返回值为 0 代表子进程*/
    {
        printf("The returned value is %d\n In child process!!\nMy PID is %d\n",result,getpid());
    }
    else /*返回值大于 0 代表父进程*/
    {
        printf("The returned value is %d\n In father process!!\nMy PID is %d\n",result,getpid());
    }
    return result;
}
```
## exec 函数族	
fork()函数是用于创建一个子进程,该子进程几乎复制了父进程的全部内容,但是,这个新创建的进程如何
执行呢?这个 exec 函数族就提供了一个在进程中启动另一个程序执行的方法。它可以根据指定的文件名或
目录名找到可执行文件,并用它来取代原调用进程的数据段、代码段和堆栈段,在执行完之后,原调用进程的内容除了进程号外,其他全部被新的进程替换了。
```
exec 函数族成员函数语法

所需头文件
#include <unistd.h>
int execl(const char *path, const char *arg, ...)
int execv(const char *path, char *const argv[])
int execle(const char *path, const char *arg, ..., char *const envp[])
int execve(const char *path, char *const argv[], char *const envp[])
int execlp(const char *file, const char *arg, ...)
int execvp(const char *file, char *const argv[])
函数返回值
-1:出错
```
exec 函数族可以默认系统的环境变量,也可以传入指定的环境变量。
```c
execlp();
/*execlp.c*/
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
if (fork() == 0)
{
/*调用 execlp()函数,这里相当于调用了"ps -ef"命令*/
if ((execlp("ps", "ps", "-ef", NULL)) < 0)
{
printf("Execlp error\n");
}
}
}

execl();
/*execl.c*/
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
if (fork() == 0)
{
/*调用 execl()函数,注意这里要给出 ps 程序所在的完整路径*/
if (execl("/bin/ps","ps","-ef",NULL) < 0)
{
printf("Execl error\n");
}
}
}

execle();
/* execle.c */
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
/*命令参数列表,必须以 NULL 结尾*/
char *envp[]={"PATH=/tmp","USER=david", NULL};
if (fork() == 0)
{
/*调用 execle()函数,注意这里也要指出 env 的完整路径*/
if (execle("/usr/bin/env", "env", NULL, envp) < 0)
{
printf("Execle error\n");
}
}
}

execve();
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
/*命令参数列表,必须以 NULL 结尾*/
char *arg[] = {"env", NULL};
char *envp[] = {"PATH=/tmp", "USER=david", NULL};
if (fork() == 0)
{
if (execve("/usr/bin/env", arg, envp) < 0)
{
printf("Execve error\n");
}
}
```

## exit()#include <stdlib.h>,_exit()#include <unistd.h>
_exit()函数的作用是:直接使进程停止运行,清除其使用的内存空间,并清除其在内核中的各种数据结构;exit()函数则在这些基础上做了一些包装,在执行退出之前加了若干道工序。exit()函数与_exit()函数最大的区别就在于 exit()函数在调用exit 系统之前要检查文件的打开情况,把文件缓冲区中的内容写回文件.

由于在 Linux 的标准函数库中,有一种被称作“缓冲 I/O(buffered I/O)”操作,其特征就是对应每一个打
开的文件,在内存中都有一片缓冲区。每次读文件时,会连续读出若干条记录,这样在下次读文件时就可
以直接从内存的缓冲区中读取;同样,每次写文件的时候,也仅仅是写入内存中的缓冲区,等满足了一定
的条件(如达到一定数量或遇到特定字符等),再将缓冲区中的内容一次性写入文件。
这种技术大大增加了文件读写的速度,但也为编程带来了一些麻烦。比如有些数据,认为已经被写入文件
中,实际上因为没有满足特定的条件,它们还只是被保存在缓冲区内,这时用_exit()函数直接将进程关闭,
缓冲区中的数据就会丢失。因此,若想保证数据的完整性,就一定要使用 exit()函数。
```c
这两个示例比较了 exit()和_exit()两个函数的区别。由于 printf()函数使用的是缓冲 I/O 方式,该函数在遇到
“\n”换行符时自动从缓冲区中将记录读出。示例中就是利用这个性质来进行比较的。
/* exit.c */
#include <stdio.h>
#include <stdlib.h>
int main()
{
printf("Using exit...\n");
printf("This is the content in buffer");
exit(0);
}
$./exit
Using exit...
This is the content in buffer $

/* _exit.c */
#include <stdio.h>
#include <unistd.h>
int main()
{
    printf("Using _exit...\n");
    printf("This is the content in buffer"); /* 加上回车符之后结果又如何 */
    _exit(0);
}
$ ./_exit
Using _exit...
$
```
## wait()和 waitpid()	 #include <sys/types.h>
#include <sys/wait.h>
wait()函数是用于使父进程(也就是调用 wait()的进程)阻塞,直到一个子进程结束或者该进程接到了一个
指定的信号为止。如果该父进程没有子进程或者他的子进程已经结束,则 wait()就会立即返回。
waitpid()的作用和 wait()一样,但它并不一定要等待第一个终止的子进程,它还有若干选项,如可提供一个
非阻塞版本的 wait()功能,也能支持作业控制。
```c
wait()
所需头文件 
#include <sys/types.h>
#include <sys/wait.h>
函数原型 pid_t wait(int *status)
函数传入值 这里的 status 是一个整型指针,是该子进程退出时的状态
 status 若不为空,则通过它可以获得子进程的结束状态
另外,子进程的结束状态可由 Linux 中一些特定的宏来测定
函数返回值 
成功:已结束运行的子进程的进程号
失败:-1

所需头文件
#include <sys/types.h>
#include <sys/wait.h>

waitpid()函数语法
函数原型
pid_t waitpid(pid_t pid, int *status, int options)

函数传入值:
pid:
pid > 0:只等待进程 ID 等于 pid 的子进程,不管已经有其他子进程运行结束退出了,只要指定的子进程还没有结束,waitpid()就会一直等下去
pid = -1:等待任何一个子进程退出,此时和 wait()作用一样
pid = 0:等待其组 ID 等于调用进程的组 ID 的任一子进程
pid < -1:等待其组 ID 等于 pid 的绝对值的任一子进程
status:同 wait()
options:
WNOHANG:若由 pid 指定的子进程不立即可用,则 waitpid()不阻塞, 此时返回值为 0
WUNTRACED:若实现某支持作业控制,则由 pid 指定的任一子进程状态已暂停,且其状态自暂停以来还未报告过,则返回其状态
0:同 wait(),阻塞父进程,等待子进程退出

函数返回值:
正常:已经结束运行的子进程的进程号
使用选项 WNOHANG 且没有子进程退出:0
调用出错:-1

/* waitpid.c */
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
    pid_t pc, pr;
    pc = fork();
    if (pc < 0)
    {
        printf("Error fork\n");
    }
    else if (pc == 0) /*子进程*/
    {
/*子进程暂停 5s*/
        sleep(5);
/*子进程正常退出*/
        exit(0);
    }
    else /*父进程*/
    {
/*循环测试子进程是否退出*/
        do
        {
/*调用 waitpid,且父进程不阻塞*/
            pr = waitpid(pc, NULL, WNOHANG);
/*若子进程还未退出,则父进程暂停 1s*/
            if (pr == 0)
            {
                printf("The child process has not exited\n");
                sleep(1);
            }
        } while (pr == 0);
/*若发现子进程退出,打印出相应情况*/
        if (pr == pc)
        {
            printf("Get child exit code: %d\n",pr);
        }
        else
        {
            printf("Some error occured.\n");
        }
    }
}

$./waitpid
The child process has not exited
The child process has not exited
The child process has not exited
The child process has not exited
The child process has not exited
Get child exit code: 75
```

## Linux 守护进程
守护进程,也就是通常所说的 Daemon 进程,是 Linux 中的后台服务进程。它是一个生存期较长的进程,
通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。守护进程常常在系统引导载
入时启动,在系统关闭时终止。Linux 有很多系统服务,大多数服务都是通过守护进程实现的.
1. 4 个步骤:

1.创建子进程,父进程退出<br>
2.在子进程中创建新会话	<br>
3.改变当前目录为根目录<br>
4.重设文件权限掩码	<br>
5.关闭文件描述符<br>


1. 创建子进程,父进程退出(孤儿进程,自动由 1 号进程(也就是 init 进程)
收养它):

由于守护进程是脱离控制终端的,因此,完成第一步后就会在 shell 终端里造成一种程序已经运行完毕的假象。之后的所有工作都在子进程中完成,而用户在 shell 终端里则可以执行其他的命令,从而在形式上做到了与控制终端的脱离。

2. 在子进程中创建新会话	

在这里使用的是系统函数 setsid(),在具体介绍 setsid()之前,读者首先要了解两个概念:进程组和会话期。

 进程组。
进程组是一个或多个进程的集合。进程组由进程组 ID 来惟一标识。除了进程号(PID)之外,进程组 ID
也是一个进程的必备属性。
每个进程组都有一个组长进程,其组长进程的进程号等于进程组 ID。且该进程 ID 不会因组长进程 的退
出而受到影响。

 会话期
会话组是一个或多个进程组的集合。通常,一个会话开始于用户登录,终止于用户退出,在此期间该用户运行的所有进程都属于这个会话期.

(1)setsid()函数作用。
setsid()函数用于创建一个新的会话,并担任该会话组的组长。调用setsid()有下面的 3 个作用。

 让进程摆脱原会话的控制。
 让进程摆脱原进程组的控制。
 让进程摆脱原控制终端的控制。

在调用 fork()函数时,子进程全盘复制了父进程的会
话期、进程组和控制终端等,虽然父进程退出了,但原先的会话期、进程组和控制终端等并没有改变,因
此,还不是真正意义上的独立,而 setsid()函数能够使进程完全独立出来,从而脱离所有其他进程的控制。
```c
#include <sys/types.h>
#include <unistd.h>
pid_t setsid(void)
成功:该进程组 ID
出错:-1
```

3. 改变当前目录为根目录	

使用 fork()创建的子进程继承了父进程的当前工作目录。由于在进程运行过程中,
当前目录所在的文件系统(比如“/mnt/usb”等)是不能卸载的,这对以后的使用会造成诸多的麻烦(比
如系统由于某种原因要进入单用户模式)。因此,通常的做法是让“/”作为守护进程的当前工作目录,这
样就可以避免上述的问题,当然,如有特殊需要,也可以把当前工作目录换成其他的路径,如/tmp。改变
工作目录的常见函数是 chdir()。

4. 重设文件权限掩码	
文件权限掩码是指屏蔽掉文件权限中的对应位。比如,有一个文件权限掩码是 050,它就屏蔽了文件组拥
有者的可读与可执行权限。由于使用 fork()函数新建的子进程继承了父进程的文件权限掩码,这就给该子进程使用文件带来了诸多的麻烦。因此,把文件权限掩码设置为 0,可以大大增强该守护进程的灵活性。
设置文件权限掩码的函数是 umask()。在这里,通常的使用方法为 umask(0)。

5. 关闭文件描述符
同文件权限掩码一样,用 fork()函数新建的子进程会从父进程那里继承一些已经打开了的文件。这些被打
开的文件可能永远不会被守护进程读或写,但它们一样消耗系统资源,而且可能导致所在的文件系统无法
被卸载。在上面的第二步之后,守护进程已经与所属的控制终端失去了联系。因此从终端输入的字符不可能达到守
护进程,守护进程中用常规方法(如 printf())输出的字符也不可能在终端上显示出来。所以,文件描述符
为 0、1 和 2 的 3 个文件(常说的输入、输出和报错这 3 个文件)已经失去了存在的价值,也应被关闭。
通常按如下方式关闭文件描述符:
```c
for(i = 0; i < MAXFILE; i++)
{
    close(i);
}
```
summary:
```c
/* daemon.c 创建守护进程实例 */
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<fcntl.h>
#include<sys/types.h>
#include<unistd.h>
#include<sys/wait.h>
int main()
{
    pid_t pid;
    int i, fd;
    char *buf = "This is a Daemon\n";

    pid = fork(); /* 第一步 */
    if (pid < 0)
    {
        printf("Error fork\n");
        exit(1);
    }
    else if (pid > 0)
    {
        exit(0); /* 父进程推出 */
    }
    setsid(); /*第二步*/
    chdir("/"); /*第三步*/
    umask(0); /*第四步*/
    for(i = 0; i < getdtablesize(); i++) /*第五步*/
    {
    close(i);
    }

    /*这时创建完守护进程,以下开始正式进入守护进程工作*/
    while(1)
    {
        if ((fd = open("/tmp/daemon.log",
O_CREAT|O_WRONLY|O_APPEND, 0600)) < 0)
        {
            printf("Open file error\n");
            exit(1);
        }
        write(fd, buf, strlen(buf) + 1);
        close(fd);
        sleep(10);
    }
    exit(0);
}
```
## 守护进程的出错处理
由于守护进程完全脱离了控制终端,因此,不能像其他普通进程一样将错误信息输出到控制终端来通知程序员,即使使用 gdb 也无法正常调试。<br>
一种通用的办法是使用 syslog 服务,将程序中的出错信息输入到系统日志文件中
(例如:“/var/log/messages”),从而可以直观地看到程序的问题所在。<br>
syslog 是 Linux 中的系统日志管理服务,通过守护进程 syslogd 来维护。该守护进程在启动时会读一个配置
文件“/etc/syslog.conf”。该文件决定了不同种类的消息会发送向何处。例如,紧急消息可被送向系统管理
员并在控制台上显示,而警告消息则可被记录到一个文件中。该机制提供了 3 个 syslog 相关函数,分别为 openlog()、syslog()和 closelog()。
```c
#include <syslog.h>
void openlog (char *ident, int option , int facility)
ident 要向每个消息加入的字符串,通常为程序的名称
facility:指定程序发送的消息类型

void syslog(int priority, char *format, ...)
priority: 指定消息的重要性
format: 以字符串指针的形式表示输出的格式,类似 printf 中的格式

void closelog(void)

/* syslog_daemon.c 利用 syslog 服务的守护进程实例 */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>
#include <syslog.h>
int main()
{
    pid_t pid, sid;
    int i, fd;
    char *buf = "This is a Daemon\n";
    pid = fork(); /* 第一步 */
    if (pid < 0)
    {
        printf("Error fork\n");
        exit(1);
    }
    else if (pid > 0)
    {
        exit(0); /* 父进程推出 */
    }

/* 打开系统日志服务,openlog */
    openlog("daemon_syslog", LOG_PID, LOG_DAEMON);

    if ((sid = setsid()) < 0) /*第二步*/
    {
        syslog(LOG_ERR, "%s\n", "setsid");
        exit(1);
    }
    if ((sid = chdir("/")) < 0) /*第三步*/
    {
        syslog(LOG_ERR, "%s\n", "chdir");
        exit(1);
    }
    umask(0); /*第四步*/
    for(i = 0; i < getdtablesize(); i++) /*第五步*/
    {
        close(i);
    }
/*这时创建完守护进程,以下开始正式进入守护进程工作*/
    while(1)
    {
        if ((fd = open("/tmp/daemon.log",O_CREAT|O_WRONLY|O_APPEND, 0600))<0)
        {
            syslog(LOG_ERR, "open");
            exit(1);
        }
        write(fd, buf, strlen(buf) + 1);
        close(fd);
        sleep(10);
    }
    closelog();
    exit(0);
}
```