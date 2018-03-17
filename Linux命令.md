# Linux命令
##  用户系统相关命令 
### 1．用户切换（su）
su zyc
-，-l，--login 为该使用者重新登录，大部分环境变量（如 HOME、SHELL 和 USER 等）和工作目 录都是以该使用者（USER）为主。若没有指定 USER，缺省情况是 root 

-m，-p 执行 su 时不改变环境变量 

-c，--command 变更账号为 USER 的使用者，执行指令（command）后再变回原来使用者 
### 2．用户管理（useradd 和 passwd）	
```
useradd 添加用户账号 useradd [选项] 用户名 
usermod 设置用户账号属性 usermod [选项] 属性值 
userdel 删除对应用户账号 userdel [选项] 用户名 
groupadd 添加组账号 groupadd [选项] 组账号 
groupmod 设置组账号属性 groupmod [选项] 属性值 
groupdel 删除对应组账号 groupdel [选项] 组账号 
passwd 设置账号密码 passwd [对应账号] 
id 显示用户 ID、组 ID 和用户所属的组列表 id [用户名] 
groups 显示用户所属的组 groups [组账号] 
who 显示登录到系统的所有用户 who 

选项 参 数 含 义 
-g  指定用户所属的群组 
-m  自动建立用户的登入目录 -n 取消建立以用户名称为名的群组
```
### 3．系统管理命令（ps 和 kill）	
```
命    令 命 令 含 义 格    式 
ps 显示当前系统中由该用户运行的进程列表 ps [选项] 
top 动态显示系统中运行的程序（一般为每隔 5s） top 
kill 输出特定的信号给指定 PID（进程号）的进程 kill [选项] 进程号（PID） 
uname 显示系统的信息（可加选项-a） uname [选项] 
setup 系统图形化界面配置 setup 
crontab 循环执行例行性命令 crontab [选项] 
shutdown 关闭或重启 Linux 系统 shutdown [选项] [时间] 
uptime 显示系统已经运行了多长时间 uptime 
clear 清除屏幕上的信息 clear 

ps:
-ef 查看所有进程及其 PID（进程号）、系统时间、命令详细目录、执行者等 
-aux 除可显示-ef 所有内容外，还可显示 CPU 及内存占用率、进程状态 
-w 显示加宽并且可以显示较多的信息 

kill:
-s 将指定信号发送给进程 
-p 打印出进程号（PID），但并不送出信号 
```
### 4．磁盘相关命令（fdisk）
```
free 查看当前系统内存的使用情况 free [选项] 
df 查看文件系统的磁盘空间占用情况 df [选项] 
du 统计目录（或文件）所占磁盘空间的大小 du [选项] 
fdisk 查看硬盘分区情况及对硬盘进行分区管理 fdisk [-l] 
```
### 5．文件系统挂载命令（mount）
### 其他指令
chown 和 chgrp	
```
（1）
① chown：修改文件所有者和组别。 
② chgrp：改变文件的组所有权。 
（2）格式。 
① chown：chown [选项]...文件所有者[所有者组名] 文件 其中的文件所有者为修改后的文件所有者。 ② chgrp：chgrp [选项]... 文件所有组 文件 

 chown root uClinux-dist.tar 
```
### chmod
chmod a+rx,u+w uClinux20031103.tgz 
<br>chmod 765 genromfs-0.5.1.tar.gz 
### grep
```
（1）作用。 在指定文件中搜索特定的内容，并将含有这些内容的行标准输出。 
（2）格式。 grep [选项] 格式 [文件及路径] 其中的格式是指要搜索的内容格式，若缺省“文件及路径”则默认表示在当前目录下搜索。 
-c 只输出匹配行的计数 
-I 不区分大小写（只适用于单字符） 
-h 查询多文件时不显示文件名 
-l 查询多文件时只输出包含匹配字符的文件名 
-n 显示匹配行及行号 
-s 不显示不存在或无匹配文本的错误信息 
-v 显示不包含匹配文本的所有行 
```
### 2.2  Linux 启动过程详解 
用户开机启动 Linux 过程如下： （1）当用户打开 PC（intel CPU）的电源时，CPU 将自动进入实模式，并从地址 0xFFFF0000 开始自 动执行程序代码，这个地址通常是 ROM-BIOS 中的地址。这时 BIOS 进行开机自检，并按 BIOS 中设 置的启动设备（通常是硬盘）进行启动，接着启动设备上安装的引导程序 lilo 或 grub 开始引导 Linux （也就是启动设备的第一个扇区），这时，Linux 才获得了启动权。 （2）第二阶段，Linux 首先进行内核的引导，主要完成磁盘引导、读取机器系统数据、实模式和保护模式 的切换、加载数据段寄存器以及重置中断描述符表等。 （3）第三阶段执行 init 程序（也就是系统初始化工作），init 程序调用了 rc.sysinit 和 rc 等程序，而 rc.sysinit 和 rc 在完成系统初始化和运行服务的任务后，返回 init。 （4）第四阶段，init 启动 mingetty，打开终端供用户登录系统，用户登录成功后进入了 shell，这样就完成 了从开机到登录的整个启动过程。 