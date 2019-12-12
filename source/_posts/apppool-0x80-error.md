---
layout: post
title: 应用程序池0x80大量报错解决办法
published: true
author: xiemx
comments: true
date: 2015-09-29 02:09:58
tags: [ iis, windows, debug ]
categories:
    - iis
    - windows
---
为应用程序池提供服务的进程意外终止。进程ID是。进程退出代码是'0x80':
```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\W3SVC\Parameters键下新建一个DWORD项，名字为：UseSharedWPDesktop值为1 重启IIS

```
