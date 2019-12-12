---
layout: post
title: apachetop – 展示web服务器实时统计数据
published: true
author: xiemx
comments: true
date: 2016-08-15 04:08:47
tags: [ apache, command ]
categories:
    - apache
---

apachetop – 展示web服务器实时统计数据

Apachetop展示Apache web服务器上关于http请求的实时表。

它显示统计（stats）, 点击（hits）, 请求（requests）, 请求细节（request details），且能够获得当前你的web服务器正在运行程序的概貌，这一点很赞。

如果你使用的是Nginx，也有一些相似的工具，但似乎没有apachetop那么详细。

安装该命令并尝试运行：
```shell
$ sudo apt-get install apachetop 
```
截图如下：

![img](/images/img_57b1811567e55.png)