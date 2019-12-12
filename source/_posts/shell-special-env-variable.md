---
layout: post
title: shell 特殊变量
published: true
author: xiemx
comments: true
date: 2016-06-29 09:06:36
tags: [ shell, linux]
categories:
    - shell
    - linux
# permalink: '/2016/06/29/shell-special-env-variable'
---
特殊变量
``` 
$#    表示变量的个数，常用于循环
$@    当前命令行所有参数
$*    当前命令行所有参数，将命令行所有参数当一个单独参数获取
$?    表示上一个命令退出的状态
$$    表示当前进程编号
$0    表示当前程序名称
$!    表示最近一个后台命令的进程编号
$IFS    表示内部的字段分隔符

$?的参考值
0    成功退出
>0    退出失败
1-125    命令退出失败，失败返回的相关值由程序定义（譬如，程序内退出只执行 exit 2，则返回为2）
126    命令找到了，但无法执行
127    命令找不到
>128    命令因受到信号而死亡
```