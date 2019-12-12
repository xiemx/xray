---
layout: post
title: Linux计划任务
published: true
author: xiemx
comments: true
date: 2015-11-28 11:11:05
tags: [ linux, command ]
categories:
    - linux
---
计划任务种类：
* 一次性计划任务（at）：由进程atd守护
* 周期性计划任务（cron）：由进程crond守护

#### 一次性计划任务（at）

生成的文件存放在／var/spool/at/目录下，任务执行一次之后自动删除。一次性计划任务还可以用batch命令去执行。batch命令会在系统空闲的情况下执行任务，用法和at相同但优先级低延时执行。

命令：
```
［root@localhost ~]# at  now + 5 minutes
  at> /bin/mail  root -s "testing at job" < /root/.bashrc
  at> <EOF>         <==输入 [ctrl] + d 结束编辑！此时会在／var/spool/at/目录下生成一个任务文件

[root@localhost ~]# atq
5 2015-11-25 23:00 a root
[root@localhost ~]# atrm 5
```
一次性计划任务的时间写法支持很多可以查看man page！当我们制定计划任务之后，由于种种原因而要去取消计划任务我们可以使用atq去查询当前系统中有多少为执行的计划，每个计划的编号是多少。在通过atrm＋任务编号来删除这个任务。／etc/at.allow和／etc/at.deny文件规范了哪些用户可以使用at那些用户无法使用at。写在allow文件表明了写在此文件中的用户才可以使用at，deny文件中的用户无法使用。

#### 周期性计划任务（cron）

由进程crond守护，生成的任务文件存放在／var/spool/cron/目录下文件会以用户名命名。也采用了／etc/cron.allow和／etc/cron.deny的授权方式。配置文件为／etc/crontab 。周期性计划任务分为两种，一种是用户级别（通过crontab －e来制定），一种为系统级别（写在／etc/crontab文件中），建议使用crontab来制定。

命令用法
```
crontab   参数
-u ：只有 root 才能进行这个任务，亦即帮其他使用者创建/移除 crontab 工作排程；
-e ：编辑 crontab 的工作内容，同vi编辑文件相同（其实就是vi了一个新文件到／var/spool/cron下）
-l ：查阅 crontab 的工作内容
-r ：移除所有的 crontab 的工作内容，若仅要移除一项，请用 -e 去编辑。

［root@localhost ～］#crontab  -u  testuser  -e    #为testuser用户制定计划任务，会打开vi的编辑界面，每一行就是一个任务
```
例：
```
###时间的写法基本就是"," "-" "/"三种符号来间隔,“＊”代表所有都匹配。
30 * * * *           <==每小时的30分，输出hello
*/10 ＊ *  *  *      <==每隔10分钟
30 12 * * 1,3,5      <==每周1 3 5的12时30分
30 12 1 3-5 *        <==每年的3 4 5月1日12时30分
```

同样的／etc/crontab 中的时间写法也是类似但`语法不同`cron每分钟都会去读一次计划任务同时也会读一次／etc/crontab文件，crontab文件中设置了4个文件夹（配置文件中的`run-parts`命令部分），系统会在不同的时间去读取运行其中的文件，我们可以将脚本文件放到对应的文件夹下也可以实现脚本的周期性执行。我们的locate数据库同步，logrotate等都是放在这些目录下来实现周期性的工作。
```
[root@localhost ~]# cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# run-parts
01 * * * * root run-parts /etc/cron.hourly <==每小时执行一次
02 4 * * * root run-parts /etc/cron.daily <==每天执行一次
22 4 * * 0 root run-parts /etc/cron.weekly <==每周日执行一次
42 4 1 * * root run-parts /etc/cron.monthly <==每个月 1 号执行一次

###run-parts 我们可以通过which命令来查看其实是/usr/bin/run-parts命令，我们可以man一下这个命令，或者直接打开这个命令的脚本，会发现这个命令会将目录下的脚本全部执行一次。
```
