---
layout: post
title: 2003系统控制面板打不开解决办法
published: true
author: xiemx
comments: true
date: 2015-09-29 03:09:32
tags: [ windows, debug ]
categories:
    - windows
---
```
运行regedit打开注册表
把 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Nls\Locale下的两个项修改为
"(Default)"="00000409"
"00000804"="1"
```