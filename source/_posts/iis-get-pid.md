---
layout: post
title: iis7如何查看进程号对应的网站
published: true
author: xiemx
comments: true
date: 2017-01-17 05:01:45
tags: [ windows, iis, webserver ]
categories:
    - windows
    - iis
---
1.获取pid

![img](/images/img_587ddd23c02a3.png)

2.查询程序池对应的pid，根据applicationpool来查找对应网站

```
%windir%/system32/inetsrv/appcmd list wp 
```

![img](/images/img_587dde19b6b47.png)