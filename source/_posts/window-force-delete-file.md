---
layout: post
title: window系统文件或目录无法删除解决方法
published: true
author: xiemx
comments: true
date: 2015-10-08 04:10:01
tags: [ windows, debug ]
categories:
    - windows
#permalink: '/2015/10/08/window-force-delete-file'
---
1. 新建.BAT批处理文件输入如下命令，然后将要删除的文件拖放到批处理文件图标上即可删除。

```powershell
DEL /F /A /Q
RD /S /Q
```

2. 可利用软件来解锁占用文件的进程推荐unlock

