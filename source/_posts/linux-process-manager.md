---
layout: post
title: Linux进程管理
published: true
author: xiemx
comments: true
date: 2015-11-15 04:11:14
tags: [ linux, tool]
categories:
    - linux
---
当我们运行程序时，Linux会为程序创建一个特殊的环境，该环境包含程序运行需要的所有资源，以保证程序能够独立运行，不受其他程序的干扰。
  
在系统中我们调用一个指令，Linux 就会创建一个新的进程。例如使用 ls 命令遍历目录中的文件时，就创建了一个进程。进程就是程序的实例。
  
系统通过一个五位数字跟踪程序的运行状态，这个数字称为 pid 或进程ID。每个进程都拥有唯一的 pid。两个 pid 一样的进程不能同时存在，Linux用 pid 来跟踪程序的运行状态。

### 进程查看
  
#### ps
```
[root@localhost ~]# ps aux
USER PID  %CPU   %MEM    VSZ    RSS    TTY    STAT    START   TIME   COMMAND
root      1       0.0        0.0        19356   1540     ?        Ss         02:09         0:01      /sbin/init
root      2      0.0         0.0        0           0           ?        S           02:09         0:00     [kthreadd]
root      3      0.0         0.0        0           0           ?        S           02:09         0:00     [migration/0]

[root@localhost ~]# ps -ef
UID     PID     PPID    C    STIME    TTY    TIME       CMD
root      1         0            0     02:09        ?      00:00:01     /sbin/init
root     2         0            0     02:09        ?      00:00:00     [kthreadd]
root     3         2            0     02:09        ?      00:00:00     [migration/0]

[root@localhost ~]# ps -le
F  S   UID PID PPID  C   PRI   NI   ADDR   SZ       WCHAN   TTY   TIME         CMD
4  S     0      1       0       0   80      0       -          4839    poll_s       ?         00:00:01    init
1  S     0      2       0       0   80      0       -           0          kthrea       ?         00:00:00   kthreadd
1  S     0      3       2       0   -40      -       -           0          migrat      ?         00:00:00    migration/0

USER 用户名 
UID 用户ID（User ID） 
PID 进程ID（Process ID） 
PPID 父进程的进程ID（Parent Process id） 
SID 会话ID（Session id） 
%CPU 进程的cpu占用率 
%MEM 进程的内存占用率 
VSZ 进程所使用的虚存的大小（Virtual Size） 
RSS 进程使用的实际内存的大小 
TTY 与进程关联的终端（tty） 
STAT 进程的状态：进程状态使用字符表示的（STAT的状态码） 
START 进程启动时间和日期 
TIME 进程使用的总cpu时间 
COMMAND 正在执行的命令行命令 
NI 优先级(Nice) 
PRI 进程优先级编号(Priority)
进程状态（S字段）： D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程

```

#### top
```
[root@localhost ~]# top
top - 02:45:58 up 36 min, 4 users, load average: 0.00, 0.00, 0.00
Tasks: 175 total, 1 running, 174 sleeping, 0 stopped, 0 zombie
Cpu(s): 0.3%us, 0.3%sy, 0.0%ni, 99.3%id, 0.0%wa, 0.0%hi, 0.0%si, 0.0%st
Mem: 1907580k total, 528968k used, 1378612k free, 26060k buffers
Swap: 2097144k total, 0k used, 2097144k free, 173020k cached
PID   USER   PR   NI    VIRT     RES    SHR     S    %CPU %MEM TIME+     COMMAND
2603  root      20    0      15028   1372    1000    R   0.3       0.1         0:00.21     top
1          root      20    0     19356    1540    1228    S    0.0      0.1          0:01.04     init
2         root      20    0      0            0          0           S   0.0       0.0        0:00.00    kthreadd


第一行显示系统运行时间，用户数，系统负载1分钟 ，5分钟，15分钟的情况
第二行显示经常总数，不同stat下的进程数量 
第三行显示cpu使用情况 第四行显示物理内存使用情况 第五行显示swap交换分区使用情况
```

#### 信号
  ```
[root@localhost ~]# kill -l
1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP
6) SIGABRT 7) SIGBUS 8) SIGFPE 9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT 17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO 30) SIGPWR
31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3
38) SIGRTMIN+4 39) SIGRTMIN+5 40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8
43) SIGRTMIN+9 44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9 56) SIGRTMAX-8 57) SIGRTMAX-7
58) SIGRTMAX-6 59) SIGRTMAX-5 60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2
63) SIGRTMAX-1 64) SIGRTMAX
```
Linux中信号有64种，通过kill命令我们可以对进程发送这些信号来告知这些进程，做什么操作。常用的以下四种
  
1) SIGHUP
这个信号用于通知它重新读取配置文件但不关闭进程，相当于reload。
2) SIGINT
程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。
9) SIGKILL
用来立即结束程序的运行. 本信号不能被阻塞、处理和忽略。如果管理员发现某个进程终止不了，可尝试发送这个信号
15) SIGTERM
程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。
  例：kill -1 2205 ——告知PID为2205的进程重载下配置文件 kill -9 2205 ——直接杀死2205进程 kill -15 2205 ——通知2205进程关闭，进程会在收到信号后结束工作后自行关闭，有些进程会屏蔽15信号如bash，此时我们只能通过9信号强制结束。

#### 进程优先级
  
假设系统中有多个进程同时在排队是，cpu应该有限处理哪个进程哪？这就取决于进程的优先级，优先级高的优先处理。进程在创建是我们可以通过nice命令指定他的优先级，对于已经开始的进程我们可以通过renice来修改他的优先级。
  
优先级范围：-20~19 数字越小优先级越高，默认创建的进程优先级为0。

例：
```
nice -n 10 vim —— 创建一个优先级为10的vim进程
[root@localhost ~]# ps -el | grep vim
F S UID PID PPID C PRI NI ADDR SZ WCHAN TTY TIME CMD
0 T 0 6891 2563 0 90 10 - 35930 signal pts/2 00:00:00 vim

renice -n -10 6891 ——调整6891进程的有限级为-10
[root@localhost ~]# renice -n -10 6891
6891: old priority 10, new priority -10

root用户可以自由调整经常的优先级，普通用户只能调低不能调高。
  
[testuser@localhost root]$ renice -n 5 6925
6925: old priority 0, new priority 5
  
[testuser@localhost root]$ renice -n 2 6925
renice: 6925: setpriority: Permission denied
```
#### 前台进程/后台进程
  
由于一个终端在处理工作时只能处理一个，当我们想在处理第二个进程时会由于终端被占用而无法处理。这时我们可以将前台的这个进程放到后台去运行，把前台的终端空出让我们运行新的进程。
  
命令：
* command &让进程在后台运行
* jobs –l 查看后台运行的进程
* fg %n 让后台运行的进程n到前台来
* bg %n 让后台的进程n运行
  
以上的%n是通过jobs -l 查看到的工作编号不是pid。

例：
```
vim & ——在后台运行vim进程
jobs -l ——查看后台所有进程
fg %1 ——将后台1号进程移动到前台来运行
bg %2 —— 让后台的2号进程开始运行
ctrl+z ——暂停当前的进程放到后台，但不运行

[root@localhost ~]# vim &
[2] 6987
  
[root@localhost ~]# vi /etc/resolv.conf
[3]+ Stopped vi /etc/resolv.conf

[root@localhost ~]# jobs -l
[1] 6891 Stopped nice -n 10 vim
[2]- 6987 Stopped (tty output) vim
[3]+ 6989 Stopped vi /etc/resolv.conf

[root@localhost ~]# bg %2
[2]- vim &

[root@localhost ~]# fg %3
```