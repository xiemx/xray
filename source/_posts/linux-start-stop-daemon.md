---
layout: post
title: start-stop-daemon 用法
published: true
author: xiemx
comments: true
date: 2017-01-07 11:01:59
tags: [ linux ]
categories:
    - linux
---
start-stop-daemon，用来启动停止系统中的守护进程，/etc/init.d/下的脚本中会用到这个命令去启动、停止、查询这个命令。可以查询学习下高玩的用法。
```
参数：
-S|--start -- <argument> ... 开启一个系统守护程序，并传递参数给它 
-K|--stop 停止一个程序 
-T|--status 得到程序的状态 
-H|--help 显示帮助信息 
-V|--version 打印版本信息 
Matching options (at least one is required): 
-p|--pidfile <pid-file> pid file to check 
-x|--exec <executable> program to start/check if it is running 
-n|--name <process-name> process name to check 
-u|--user <username|uid> process owner to check 
Options: 
-g|--group <group|gid> 按指定用户组权限运行程序 
-c|--chuid <name|uid[:group|gid]> 
按指定用户、用户组权限运行程序 
-s|--signal <signal> signal to send (default TERM) 
-a|--startas <pathname> program to start (default is <executable>) 
-r|--chroot <directory> chroot to <directory> before starting 
-d|--chdir <directory> change to <directory> (default is /) 
-N|--nicelevel <incr> add incr to the process' nice level 
-P|--procsched <policy[:prio]> 
use <policy> with <prio> for the kernel 
process scheduler (default prio is 0) 
-I|--iosched <class[:prio]> use <class> with <prio> to set the IO 
scheduler (default prio is 4) 
-k|--umask <mask> 在开始运行前设置<mask> 
-b|--background 后台运行 
-m|--make-pidfile 当命令本身不创建pidfile时，由start-stop-daemon创建 
-R|--retry <schedule> 等待timeout的时间，检查进程是否停止，如果没有发送KILL信号； 
-t|--test 测试模式 
-o|--oknodo exit status 0 (not 1) if nothing done 
-q|--quiet 不要输出警告 
-v|--verbose 显示运行过程信息 
```

开启一个daemon进程 
```
start-stop-daemon --start --background --exec /test.py  -- -c /test.conf  
# -- 后的参数会传递给 /test.py,相当于/test.py -c /test.conf
```

关闭一个daemon进程 
```
start-stop-daemon --stop --name proxy.py
```