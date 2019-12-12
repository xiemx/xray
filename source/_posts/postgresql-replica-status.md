---
layout: post
title: PostgreSQL查看复制状态
published: true
author: xiemx
comments: true
date: 2019-07-08 11:07:36
tags: [ postgresql, database ]
categories:
    - postgresql
# permalink: '/2019/07/08/postgresql%e6%9f%a5%e7%9c%8b%e5%a4%8d%e5%88%b6%e7%8a%b6%e6%80%81'
---
#### postgresql查看复制状态，master上执行
```sql
#select * from pg_stat_replication; 
postgres=# select * from pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 13321
usesysid         | 17019
usename          | replication
application_name | walreceiver
client_addr      | 10.0.0.81
client_hostname  | 
client_port      | 42809
backend_start    | 2016-08-11 10:57:35.856289+08
backend_xmin     | 
state            | streaming --同步状态
sent_location    | 1/E0CE9750
write_location   | 1/E0CE9750
flush_location   | 1/E0CE9750
replay_location  | 1/E0CE9750
sync_priority    | 0
sync_state       | async  --同步模式

state: 同步状态
    streaming : 同步
    startup : 连接中
    catchup: 同步中

sync_state: 同步模式.
    async : 异步
    sync : 同步
    potential: 虽然现在是异步,但有可能提升到同步
```
#### 查看复制的延迟有多少，字节单位，master上执行 
```sql
#select pg_xlog_location_diff(sent_location, replay_location) from pg_stat_replication; 

posrgresql=# select pg_xlog_location_diff(sent_location, replay_location) from pg_stat_replication; 
 pg_xlog_location_diff 
-----------------------
                      0
(1 row)
```
#### slave上查看sql滞后时间
```sql
SELECT CASE WHEN pg_last_xlog_receive_location() = pg_last_xlog_replay_location()
    THEN 0
    ELSE EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp())
    END AS log_delay;

postgres=# SELECT CASE WHEN pg_last_xlog_receive_location() = pg_last_xlog_replay_location()
postgres-#     THEN 0
postgres-#     ELSE EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp())
postgres-#     END AS log_delay;
 log_delay
-----------
         0
(1 row)
```
#### slave上查看是否处于recovery模式
```sql
select pg_is_in_recovery();
postgres=# select pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)
```
#### slave上查看最新的reploy时间戳
```sql
#select pg_last_xact_replay_timestamp();
postgres=# select pg_last_xact_replay_timestamp();
 pg_last_xact_replay_timestamp
-------------------------------
 2019-07-08 03:01:33.854131+00
(1 row)
```

#### slave上查看最新的reploy位置
```sql
#select pg_last_xlog_replay_location();
postgres=# select pg_last_xlog_replay_location();
 pg_last_xlog_replay_location
------------------------------
 220C/56EB4C10
(1 row)
```