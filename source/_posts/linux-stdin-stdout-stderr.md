---
layout: post
title: Linux标准输入输出重定向
published: true
author: xiemx
comments: true
date: 2015-11-07 05:11:17
tags: [ linux ]
categories:
    - linux
---
一个命令执行前，先会准备好所有输入输出，默认分别绑定（stdin,stdout,stderr)，如果这个时候出现错误，命令将终止，不会执行。执行成功会将结果输出到屏幕上，执行错误时也会将错误信息输出到屏幕。这些默认的输出，输入都是linux系统内定的，我们在使用过程中，有时候并不希望执行结果输出到屏幕。我想输出到文件或其它设备。这个时候我们就需要进行输出重定向了。这些标准输入输出对应的文件如下：

```
[root@localhost ~]# ll /dev/stdin /dev/stdout /dev/stderr
lrwxrwxrwx. 1 root root 15 Nov 8 01:03 /dev/stderr -> /proc/self/fd/2 
lrwxrwxrwx. 1 root root 15 Nov 8 01:03 /dev/stdin -> /proc/self/fd/0 
lrwxrwxrwx. 1 root root 15 Nov 8 01:03 /dev/stdout -> /proc/self/fd/1
```

由这些管道文件可以看到linux shell下常用输入输出操作符是：

1.  标准输入   (stdin) ：代码为 0 ，使用 < 
2.  标准输出   (stdout)：代码为 1 ，使用 > 或 >> 
3.  标准错误输出(stderr)：代码为 2 ，使用 2> 或 2>> 

当我们执行命令时我们不想要程序将结果输出到屏幕时我们就可以调用`>` `>>`重定向操作符来重定向数据流的输出位置
```shell
ls -l > testfile1  ##查看当前目录下的文件属性，但不输出到屏幕，输出到testfile1的文件中。

echo 123qwe321|passwd --stdin testuse
###输出字符串123qwe321，并将字符串通过标准输入设备传递给passwd程序去修改testuser用户的密码。
###--stdin代表标准输入设备，也可以用/dev/stdin来代替。(stdin前是双横杠)

重定向时要注意
## >重定向输出到文件中时会清空文件中的内容
## >>不会清空文件内容会在文件最后追加输出内容
###由于>会清空文件在写入输出内容，因此常用来清空文件

echo "" > /tmp/testfile1   ##清空testfile1文件。
```