---
layout: post
title: twemproxy配置文件详解
published: true
author: xiemx
comments: true
date: 2017-02-09 05:02:29
tags: [ twemproxy, redis, memcache ]
categories:
    - memcached
    - redis
# permalink: '/2017/02/09/twemproxy-configure'
---
配置文件详解：
```conf
listen
twemproxy监听的端口。可以以ip:port或name:port的形式来书写。
 
hash
可以选择的key值的hash算法：
> one_at_a_time
> md5
> crc16
> crc32(crc32 implementation compatible with libmemcached)
> crc32a(correct crc32 implementation as per the spec)
> fnv1_64
> fnv1a_64
> fnv1_32
> fnv1a_32
> hsieh
> murmur
> jenkins
如果没选择，默认是fnv1a_64。

hash_tag
hash_tag允许根据key的一个部分来计算key的hash值。hash_tag由两个字符组成，一个是hash_tag的开始，另外一个是hash_tag的结束，在hash_tag的开始和结束之间，是将用于计算key的hash值的部分，计算的结果会用于选择服务器。
 
例如：如果hash_tag被定义为”{}”，那么key值为"user:{user1}:ids"和"user:{user1}:tweets"的hash值都是基于”user1”，最终会被映射到相同的服务器。而"user:user1:ids"将会使用整个key来计算hash，可能会被映射到不同的服务器。
 
distribution
存在ketama、modula和random3种可选的配置。其含义如下：
 
	(1)ketama
	ketama一致性hash算法，会根据服务器构造出一个hash ring，并为ring上的节点分配hash范围。ketama的优势在于单个节点添加、删除之后，会最大程度上保持整个群集中缓存的key值可以被重用。
	 
	(2)modula
	modula非常简单，就是根据key值的hash值取模，根据取模的结果选择对应的服务器。
	 
	(3)random
	random是无论key值的hash是什么，都随机的选择一个服务器作为key值操作的目标。
 
timeout
单位是毫秒，是连接到server的超时值。默认是永久等待。
 
backlog
监听TCP 的backlog（连接等待队列）的长度，默认是512。
 
preconnect
是一个boolean值，指示twemproxy是否应该预连接pool中的server。默认是false。
 
redis
是一个boolean值，用来识别到服务器的通讯协议是redis还是memcached。默认是false。
 
server_connections
每个server可以被打开的连接数。默认，每个服务器开一个连接。
 
auto_eject_hosts
是一个boolean值，用于控制twemproxy是否应该根据server的连接状态重建群集。这个连接状态是由server_failure_limit 阀值来控制。
默认是false。
 
server_retry_timeout
单位是毫秒，控制服务器连接的时间间隔，在auto_eject_host被设置为true的时候产生作用。默认是30000 毫秒。
 
server_failure_limit
控制连接服务器的次数，在auto_eject_host被设置为true的时候产生作用。默认是2。
 
servers
一个pool中的服务器的地址、端口和权重的列表，包括一个可选的服务器的名字，如果提供服务器的名字，将会使用它决定server的次序，从而提供对应的一致性hash的hash ring。否则，将使用server被定义的次序
```

