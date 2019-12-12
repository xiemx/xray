---
layout: post
title: 限制进程的CPU利用率
published: true
author: xiemx
comments: true
date: 2016-07-06 10:07:20
tags: [ linux, command, cpulimit ]
categories:
    - linux
---

1. cpulimit安装

```shell
apt-get install cpulimit
yum install cpulimit
### 请先安装epel源，在执行yum命令.
```

2. cpulimit实例

```
### 根据进程ID限值
cpulimit -p 1234 -l 40
进程ID为1234的程序只能使用40%的cpu

### 根据进程路径限值
cpulimit -e nginx -l 50
nginx只能使用50%的cpu
```

3. 注意事项
这边要留意一点，-l后面默认值是百分比，而且在双核情况下要减半。例如nginx的例子，在双核cpu情况下他可以利用25%的cpu，在4核的情况下，只能使用12.5%的cpu.root用户可以限值所有的进程，普通用户只能限值自己程序.

项目地址：http://cpulimit.sourceforge.net/