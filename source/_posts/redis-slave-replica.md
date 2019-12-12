---
layout: post
title: redis主从复制
published: true
author: xiemx
comments: true
date: 2016-05-23 11:05:17
tags: [ redis ]
categories:
    - redis
# permalink: '/2016/05/23/redis%e4%b8%bb%e4%bb%8e%e5%a4%8d%e5%88%b6'
---
1. redis主从复制特点:

* master可以拥有多个slave
* 多个slave可以连接同一个master外，还可以连接到其他slave
* 主从复制不会阻塞master，在同步数据时，master可以继续处理client请求
* 提高系统的伸缩性
* 可以在master禁用数据持久化，注释掉master配置文件中的所有save配置，只需在slave上配置数据持久化

2. redis主从复制过程:

当配置好slave后，slave与master建立连接，然后发送sync命令。无论是第一次连接还是重新连接，master都会启动一个后台进程，将数据库快照保存到文件中，同时master主进程会开始收集新的写命令并缓存。后台进程完成写文件后，master就发送文件给slave，slave将文件保存到硬盘上，再加载到内存中，接着master就会把缓存的命令转发给slave，后续master将收到的写命令发送给slave。如果master同时收到多个slave发来的同步连接命令，master只会启动一个进程来写数据库镜像，然后发送给所有的slave。

3. redis主从配置:
```conf
### master
daemonize yes
pidfile /var/run/redis.pid
port 6379
timeout 300
loglevel verbose
logfile /usr/local/xiemx-redis/var/log/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
rdbcompression yes
dbfilename dump.rdb
dir /usr/local/xiemx-redis/var/data
requirepass redis
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
slowlog-log-slower-than 10000
slowlog-max-len 1024
vm-enabled no
vm-swap-file /tmp/redis.swap
vm-max-memory 0
vm-page-size 32
vm-pages 134217728
vm-max-threads 4
hash-max-zipmap-entries 512
hash-max-zipmap-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
activerehashing yes
```
```conf
### slave
daemonize yes
pidfile /var/run/redis.pid
port 6379
timeout 300
loglevel verbose
logfile /usr/local/xiemx-redis/var/log/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
rdbcompression yes
dbfilename dump.rdb
dir /usr/local/xiemx-redis/var/data
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
slowlog-log-slower-than 10000
slowlog-max-len 1024
vm-enabled no
vm-swap-file /tmp/redis.swap
vm-max-memory 0
vm-page-size 32
vm-pages 134217728
vm-max-threads 4
hash-max-zipmap-entries 512
hash-max-zipmap-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
activerehashing yes
slaveof 192.168.1.189 6379
masterauth redis
```
4. redis复制测试

```log
查看master端日志:
[8930] 31 Jul 19:16:09 - Accepted 192.168.1.136:54774
[8930] 31 Jul 19:16:09 * Slave ask for synchronization
[8930] 31 Jul 19:16:09 * Starting BGSAVE for SYNC
[8930] 31 Jul 19:16:09 * Background saving started by pid 10782
[10782] 31 Jul 19:16:09 * DB saved on disk
[8930] 31 Jul 19:16:09 * Background saving terminated with success
[8930] 31 Jul 19:16:09 * Synchronization with slave succeeded
[8930] 31 Jul 19:16:14 - DB 0: 1 keys (0 volatile) in 4 slots HT.
[8930] 31 Jul 19:16:14 - 1 clients connected (1 slaves), 807320 bytes in use

查看slave端日志:
[24398] 01 Aug 10:16:10 * Connecting to MASTER...
[24398] 01 Aug 10:16:10 * MASTER  SLAVE sync started: SYNC sent
[24398] 01 Aug 10:16:10 * MASTER  SLAVE sync: receiving 25 bytes from master
[24398] 01 Aug 10:16:10 * MASTER  SLAVE sync: Loading DB in memory
[24398] 01 Aug 10:16:10 * MASTER  SLAVE sync: Finished with success
[24398] 01 Aug 10:16:15 - DB 0: 1 keys (0 volatile) in 4 slots HT.
[24398] 01 Aug 10:16:15 - 1 clients connected (0 slaves), 798960 bytes in use
```
```
master端操作:
redis 127.0.0.1:6379> set k_m master
OK

slave端操作:
redis 127.0.0.1:6379> get k_m
"master"
```
