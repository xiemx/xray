---
layout: post
title: redis 淘汰策略
published: true
author: xiemx
comments: true
date: 2017-07-14 10:07:44
tags: [ redis ]
categories:
    - redis
# permalink: '/2017/07/14/redis-%e6%b7%98%e6%b1%b0%e7%ad%96%e7%95%a5'
---
redis 内存数据使用到maxmemory的时候，就会根据maxmemory_policy设定的淘汰策略进行内存整理数据回收。选择不同的策略，要根据redis的用途来区分。

redis 提供 6种数据淘汰策略：

* volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰
* volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰
* volatile-random：从已设置过期时间的数据集中任意选择数据淘汰
* allkeys-lru：从数据集中挑选最近最少使用的数据淘汰
* allkeys-random：从数据集中任意选择数据淘汰
* noenviction：禁止删除数据