---
layout: post
title: Linux日志系统
published: true
author: xiemx
comments: true
date: 2015-11-28 10:11:40
tags: [ linux ]
categories:
    - linux
---
在Linux系统中，有三个主要的日志子系统：

* 连接日志--由多个程序执行，把纪录写入到/var/log/wtmp和/var/run/utmp，login等程序更新wtmp和utmp文件，使系统管理员能够跟踪谁在何时登录到系统。
* 进程统计--由系统内核执行。当一个进程终止时，为每个进程往进程统计文件（pacct或acct）中写一个纪录。进程统计的目的是为系统中的基本服务提供命令使用统计。
* 错误日志--由syslogd（8）执行。各种系统守护进程、用户程序和内核通过syslog（3）向文件/var/log/messages报告值得注意的事件。另外有许多UNIX程序创建日志。像HTTP和FTP这样提供网络服务的服务器也保持详细的日志。

大部分Linux系统都使用syslog管理日志，系统日志默认会写在/var/log目录下，syslog会依据/etc/syslog.conf文件中的配置将不同级别、不同类别的日志分门别类的纪录到不同的日志文件中去。/etc/syslog.conf中依照如下格式纪录日志配置：
```
日志对象.级别                  日志文件存放位置
authpriv.info                /var/log/secure
```
```
［root@localhost ～］#cat /etc/syslog.conf

//将info或更高级别的消息送到/var/log/messages，除了mail以外。
//其中*是通配符，代表任何设备；none表示不对任何级别的信息进行记录。
*.info;mail.none;authpriv.none /var/log/messages
//将authpirv设备的任何级别的信息记录到/var/log/secure文件中，这主要是一些和认、权限使用相关的信息。
authpriv.* /var/log/secure
//将mail设备中的任何级别的信息记录到/var/log/maillog文件中，这主要是和电子邮件相关的信息。
mail.* /var/log/maillog
//将cron设备中的任何级别的信息记录到/var/log/cron文件中，这主要是和系统中定期执行的任务相关的信息。
cron.* /var/log/cron
//将任何设备的emerg级别的信息发送给所有正在系统上的用户。
*.emerg *
//将uucp和news设备的crit级别的信息记录到/var/log/spooler文件中。
uucp,news.crit /var/log/spooler
//将和系统启动相关的信息记录到/var/log/boot.log文件中。
local7.* /var/log/boot.log
```
##### 日志对象

kern——内核
User——用户程序
Damon——系统守护进程
Mail——电子邮件系统
Auth——与安全权限相关的命令
Lpr——打印机
News——新闻组信息
Uucp——Uucp程序
Cron——记录当前登录的每个用户信息
wtmp——一个用户每次登录进入和退出时间的永久记录
Authpriv——授权信息

##### 日志的级别

emerg——最高的紧急程度状态
alert——紧急状态
Cirt——重要信息
warning——警告
err——错误
notice——通知
info——一般性消息
Debug——调试级信息
None——不记录任何日志信息常用的日志文件

##### 常见的日志文件

cron: crontab例行事务的日志
dmesg: 内核启动时的检测信息,输出同 dmesg 命令
lastlog: 所有帐号最后一次登录的相关信息，输出同 lastlog 命令
maillog: 邮件来往信息
messages ： 系统错误信息
secure ： 涉及到“输入口令”的操作，都会记录于此
wtmp与faillog: 登录成功的用户信息(wtmp)和登录失败的用户信息(faillog)
httpd, samba等 ： 不同的网络服务用自己的定制的日志文件
utmp、wtmp和lastlog日志文件是多数重用UNIX日志子系统的关键--保持用户登录进入和退出的纪录。有关当前登录用户的信息记录在文件utmp中；登录进入和退出纪录在文件wtmp中；最后一次登录文件可以用lastlog命令察看。数据交换、关机和重起也记录在wtmp文件中。通常，每次有一个用户登录时，login程序在文件lastlog中察看用户的UID。如果找到了，则把用户上次登录、退出时间和主机名写到标准输出中，然后login程序在lastlog中纪录新的登录时间。在新的lastlog纪录写入后，utmp文件打开并插入用户的utmp纪录。该纪录一直用到用户登录退出时删除。utmp文件被各种命令文件使用，包括who、w、users和finger。 下一步，login程序打开文件wtmp附加用户的utmp纪录。当用户登录退出时，具有更新时间戳的同一utmp纪录附加到文件中。wtmp文件被程序last和ac使用。

### 日志轮替（logrotate）

在系统用户众多的，或系统运行有访问量非常大的程序时，日志文件的增长会非常迅速，严重的会导致系统磁盘被写满数据无法被写入系统进程报错中止等。为了应对这个问题，Linux采取了日志轮替的方式来管理日志文件。logrotate程序会依据/etc/logrotate.conf和/etc/logrotate.d/中定义的轮替规则来裁剪、删除、备份日志文件。logrotate会在/etc/cron.daily目录下生成脚本，每天自动执行。
```
[root@localhost ~]# cat /etc/logrotate.conf

weekly                        #默认每周进行一次日志清理
rotate 4                      #保留的日志文件数量
create                         #创建一个新的来存储
#compress                #是否需要压缩
include /etc/logrotate.d  #读取/etc/logrotate.d目录下的文件
/var/log/wtmp {               #针对单个wtmp日志操作
monthly                              #每个月一次
minsize 1M                         #超过1M整理
create 0664 root utmp     #新加文件权限和用户组
rotate 1                                #保留一个文件
}
```
 
