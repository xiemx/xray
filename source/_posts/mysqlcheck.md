---
layout: post
title: MySQL数据库mysqlcheck的使用方法详解
published: true
author: xiemx
comments: true
date: 2015-08-24 01:08:44
tags: [ mysql, mysqlcheck, tool ]
categories:
    - mysql
---
mysqlcheck，是mysql自带的可以检查和修复`MyISAM`表，并且它还可以优化和分析表，mysqlcheck的功能类似myisamchk，但其工作不同。主要差别是当mysqld服务器在运行时必须使用mysqlcheck，而myisamchk应用于服务器没有运行时。myisamchk修复失败是不可逆的。
```shell
1.如果需要检查并修复所有的数据库的数据表
mysqlcheck -A -o -r -p
Enter password:
database1 OK
database2 OK

2.如果需要修复指定的数据库用
mysqlcheck -A -o -r database1 -p
database1 OK
```