---
layout: post
title: nf_conntrack丢包
published: true
author: xiemx
comments: true
date: 2016-12-20 01:12:44
tags: [ linux, debug ]
categories:
    - linux
---
#### 现象

![img](/images/QQ20161220-0.png)

nf_conntrack/ip_conntrack 跟 nat 有关，用来跟踪连接条目，它会使用一个哈希表来记录 established 的记录。如果该哈希表满了，就会出现问题。

原因是nf_conntrack: table full, dropping packet，CONNTRACK_MAX 允许的最大跟踪连接条目超过了我们设置的阀值65535

#### 解决：

```
echo "net.nf_conntrack_max=655350" >> /etc/sysctl.conf&&sysctl -p 
```