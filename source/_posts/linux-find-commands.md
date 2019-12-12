---
layout: post
title: Linux查找
published: true
author: xiemx
comments: true
date: 2015-10-16 10:10:31
tags: [ linux, command ]
categories:
  - linux
---
1. find

功能强大，用法自行google

2. locate

locate命令其实是"find -name"的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/mlocate/mlocate.db），这个数据库中含 有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可 以在使用locate之前，先使用updatedb命令，手动更新数据库。

例：

locate /etc/sh* ——搜索etc目录下所有以sh开头的文件。

locate -i ~/m*——搜索用户主目录下，所有以m开头的文件，并且忽略大小写。

3. whereis

whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

4. which

which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。
