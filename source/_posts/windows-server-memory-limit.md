---
layout: post
title: 2003系统8G内存条只能识别3G解决办法
published: true
author: xiemx
comments: true
date: 2015-09-29 02:09:15
tags: [ windows, debug ]
categories:
    - windows
---
#### 现象
32位的2003系统的机器，8G内存条只能识别3G，这种情况需要进系统进行修改。具体修改如下：
win2003 32位企业版理论上能支持最大32G内存，但有些机器系统只认3.25G
#### 解决办法：
```
C：\boot.ini 修改里面的命令（\boot.ini有可能被隐藏）
default=multi(0)disk(0)rdisk(0)partition(2)\WINDOWS
[operating systems]
(0)disk(0)rdisk(0)partition(2)\WINDOWS=Windows Server 2003, Enterprise&quot; /fastdetect /PAE
```