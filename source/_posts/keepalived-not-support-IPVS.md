---
layout: post
title: keepalived不支持IPVS
published: true
author: xiemx
comments: true
date: 2016-08-24 05:08:21
tags: [ keepalived, cluster]
categories:
    - keepalived
---
编译keepalived是遇到keepalived无法支持ip_vs，情况如下：
  
keepalived默认编译时是在/usr/src/linux下找内核源代码。默认安装的源码在`/usr/src/kernels/$(uname -r)/`目录下（如果没有这个目录可以安装下kernel-devel包)。
  
解决办法：
  ```
ln -s /usr/src/kernels/`uname -r`/  /usr/src/linux
或者编译时指定 --with-kernel-dir=/usr/src/kernels/`uname -r`/
然后重新编译keepalived，现在官方的最新版已经很少出现这种情况，暂时不知道是不是版本问题。
```
