---
layout: post
title: redis repl_diskless_replication
published: true
author: xiemx
comments: true
date: 2017-03-06 04:03:55
tags: [ redis ]
categories:
    - redis
---
redis diskless replication
```conf
# Replication
role:slave
master_host:192.168.10.226
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:1
slave_repl_offset:1
master_sync_left_bytes:-4022404224
master_sync_last_io_seconds_ago:37
master_link_down_since_seconds:1488785860
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0


# Replication
role:master
connected_slaves:2
slave0:ip=192.168.10.192,port=6379,state=online,offset=1134556373255,lag=1
slave1:ip=192.168.10.82,port=6379,state=wait_bgsave,offset=0,lag=0
master_repl_offset:1134556812751
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1134555764176
repl_backlog_histlen:1048576
```



```log
6581:C 06 Mar 15:28:27.766 * DB saved on disk
26581:C 06 Mar 15:28:27.990 * RDB: 212 MB of memory used by copy-on-write
2251:M 06 Mar 15:28:28.379 * Background saving terminated with success
2251:M 06 Mar 15:29:08.687 * Synchronization with slave 192.168.10.82:6379 succeeded
2251:M 06 Mar 15:29:47.093 # Client id=314558845 addr=192.168.10.82:58002 fd=16 name= age=194 idle=1 flags=S db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=3617 omem=95797745 events=rw cmd=psync scheduled to be closed ASAP for overcoming of output buffer limits.
2251:M 06 Mar 15:29:47.093 # Connection with slave 192.168.10.82:6379 lost.
2251:M 06 Mar 15:30:15.999 * Slave 192.168.10.82:6379 asks for synchronization
2251:M 06 Mar 15:30:15.999 * Unable to partial resync with slave 192.168.10.82:6379 for lack of backlog (Slave request was: 1134354232038).
2251:M 06 Mar 15:30:15.999 * Starting BGSAVE for SYNC with target: disk
2251:M 06 Mar 15:30:16.232 * Background saving started by pid 30494
30494:C 06 Mar 15:31:52.413 * DB saved on disk
30494:C 06 Mar 15:31:52.597 * RDB: 223 MB of memory used by copy-on-write
2251:M 06 Mar 15:31:52.950 * Background saving terminated with success
2251:M 06 Mar 15:32:33.375 * Synchronization with slave 192.168.10.82:6379 succeeded
2251:M 06 Mar 15:33:08.106 # Client id=314561307 addr=192.168.10.82:38644 fd=16 name= age=173 idle=1 flags=S db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=4325 omem=102199438 events=rw cmd=psync scheduled to be closed ASAP for overcoming of output buffer limits.
2251:M 06 Mar 15:33:08.106 # Connection with slave 192.168.10.82:6379 lost.
2251:M 06 Mar 15:33:40.471 * Slave 192.168.10.82:6379 asks for synchronization
2251:M 06 Mar 15:33:40.471 * Unable to partial resync with slave 192.168.10.82:6379 for lack of backlog (Slave request was: 1134453799057).
2251:M 06 Mar 15:33:40.471 * Starting BGSAVE for SYNC with target: disk
2251:M 06 Mar 15:33:40.715 * Background saving started by pid 1440
2251:M 06 Mar 15:35:14.354 # Connection with slave 192.168.10.82:6379 lost.
1440:C 06 Mar 15:35:20.564 * DB saved on disk
1440:C 06 Mar 15:35:20.743 * RDB: 296 MB of memory used by copy-on-write
2251:M 06 Mar 15:35:21.013 * Background saving terminated with success
```


### 原理分析
如果设置了一个slave，不管是在第一次链接还是重新链接master的时候，slave会发送一个同步命令
然后master开始后台保存，收集所有对修改数据的命令。当后台保存完成，master会将这个数据文件传送到slave，
然后保存在磁盘，加载到内存中，master接着发送收集到的所有的修改数据的命令。

Redis为避免输出缓冲区过度耗用内存，使用client-output-buffer-limit参数限制客户端输出缓冲区内存使用量。
Redis数据复制过程中，Slave有个flags=S的客户端连接到Master; 它和其他客户端一样有输出缓冲区和缓冲区大小限制。
`client-output-buffer-limit slave 256mb 64mb 60`
当缓冲区使用超过256mb,Master会尽快杀掉它；
当缓冲区使用大于64mb,且小于256mb的soft limit值时，并持续时间达60秒，也会被Master尽快杀掉。


通过以上的原理分析我们可以得到一个解决方法：扩大buffer和timeout，但对于一个数据量比较大的db且服务器本身内存不足的情况下。
我们可以采用diskless replication，直接通过网络将数据更新过去，不再让服务器去同步到disk再从disk输出到内存导致output报错


### 解决
开启redis diskless replication

```shell
127.0.0.1:6379> CONFIG set repl-diskless-sync yes
OK
(1.97s)
```
```log
2251:M 06 Mar 15:35:34.896 * Slave 192.168.10.82:6379 asks for synchronization
2251:M 06 Mar 15:35:34.896 * Full resync requested by slave 192.168.10.82:6379
2251:M 06 Mar 15:35:34.896 * Delay next BGSAVE for SYNC
2251:M 06 Mar 15:35:40.203 * Starting BGSAVE for SYNC with target: slaves sockets
2251:M 06 Mar 15:35:40.469 * Background RDB transfer started by pid 3351
3351:C 06 Mar 15:37:03.206 * RDB: 233 MB of memory used by copy-on-write
2251:M 06 Mar 15:37:03.473 * Background RDB transfer terminated with success
2251:M 06 Mar 15:37:03.473 # Slave 192.168.10.82:6379 correctly received the streamed RDB file.
2251:M 06 Mar 15:37:03.473 * Streamed RDB transfer with slave 192.168.10.82:6379 succeeded (socket). Waiting for REPLCONF ACK from slave to enable streaming
2251:M 06 Mar 15:37:24.848 * 50000 changes in 60 seconds. Saving...
2251:M 06 Mar 15:37:25.097 * Background saving started by pid 5916
2251:M 06 Mar 15:38:10.510 * Synchronization with slave 192.168.10.82:6379 succeeded
5916:C 06 Mar 15:39:06.535 * DB saved on disk
5916:C 06 Mar 15:39:06.742 * RDB: 250 MB of memory used by copy-on-write
2251:M 06 Mar 15:39:07.093 * Background saving terminated with success
```
```conf
# Replication
role:slave
master_host:192.168.10.226
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:1134697190334
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# Replication
role:master
connected_slaves:2
slave0:ip=192.168.10.192,port=6379,state=online,offset=1135553694283,lag=1
slave1:ip=192.168.10.82,port=6379,state=online,offset=1135553653727,lag=1
master_repl_offset:1135554186742
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1135553138167
repl_backlog_histlen:1048576
```