---
layout: post
title: mysql master-slave
published: true
author: xiemx
comments: true
date: 2016-01-31 12:01:41
tags: [ mysql ]
categories:
    - mysql
---
Mysql复制（replication）是一个异步的复制，从一个Mysql 实例（Master）复制到另一个Mysql 实例（Slave）。实现整个主从复制，需要由Master服务器上的IO进程，和Slave服务器上的Sql进程和IO进程共从完成。要实现主从复制，首先必须打开Master端的binary log（bin-log）功能，因为整个 MySQL 复制过程实际上就是Slave从Master端获取相应的二进制日志，然后再在自己slave端完全顺序的执行日志中所记录的各种操作。 （二进制日志几乎记录了除select以外的所有针对数据库的sql操作语句）主从同步也称之为AB复制。

基本过程：

* Slave端的IO进程连接上Master，向Master请求指定日志文件的指定position后的日志内容；
* Master接收到来自Slave的IO进程的请求后，负责复制的IO进程根据Slave的请求信息，读取相应日志内容，返回给Slave的IO进程。并将本次请求读取的bin-log文件名及位置一起返回给Slave端。
* Slave的IO进程接收到信息后，将接收到的日志内容依次添加到Slave端的relay-log文件的最末端，并将读取到的Master端的bin-log的文件名和位置记录到master-info文件中，以便在下一次读取的时候能够清楚的告诉Master“我需要从某个bin-log的哪个位置开始往后的日志内容，请发给我”。
* Slave的Sql进程检测到relay-log中新增加了内容后，会解析relay-log的内容成为在Master端真实执行时候的那些可执行的内容，并在自身执行。

###数据库主从同步：
```
主：servera 172.25.30.10     
从：serverb 172.25.30.11
```

数据库的主从同步（AB复制），要保证数据的一致性，首先将主上的数据库备份出来，在从机上恢复一下保证基础的数据相同。从上述过程中我们可以得知slave是通过获取master的二进制日志来重演来达到数据同步的，因此master需要开启二进制日志，slave来获取二进制日志时使用什么账户，账号具有什么权限需要在master端定义，salve需要知道谁是master，用什么账户、同步哪个二进制文件、从二进制文件的哪个position开始同步。另外还需要设置下主从的server id。

由上推导出操作步骤

1. master/slave先设置server-id，master在开启二进制日志功能log-bin
```
[root@localhost mysql]# cat /etc/my.cnf
[mysqld]
server-id=1            server-id数字任选，但不可重复
datadir=/var/lib/mysql
log-bin=/var/log/mysql/binlog          slave不需要开启此功能
socket=/var/lib/mysql/mysql.sock
user=mysql
log-slow-queries=/var/log/mysql/slowlog
symbolic-links=0
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```
2. master创建同步用户，并授权
```sql
mysql>grant   replication  slave on *.*  to xiemx@‘172.25.30.%’ identified by '123123';flush privileges;
```
3. slave端指定master信息
```
mysql>change master to master_host='172.25.30.10',master_user='xiemx',master_password='123123',master_log_file="binlog.000003"，master_log_pos=245;

mysql>slave start；
```
4. 查看salve状态信息；
```sql
mysql> show slave status \G;           \G表示将行和列翻转过来显示
*************************** 1. row ***************************
Slave_IO_State: Waiting for master to send event
Master_Host: 172.25.30.11
Master_User: xiemx
Master_Port: 4331
Connect_Retry: 60
Master_Log_File: binlog.000003
Read_Master_Log_Pos: 245
Relay_Log_File: mysqld-relay-bin.000001           中继日志
Relay_Log_Pos: 245
Slave_IO_Running: Yes               slave的io是否正常运行
Slave_SQL_Running: Yes              slave的sql是否正常运行，需要注意两个都为yes也不一定正常。但有一个no肯定不正常。我们需要在测试下读写。
Replicate_Do_DB:
Replicate_Ignore_DB: mysql
Replicate_Do_Table:
Replicate_Ignore_Table:
Replicate_Wild_Do_Table: photo.%
Replicate_Wild_Ignore_Table: mysql.%
Last_Errno: 0
Last_Error:
Skip_Counter: 0
Exec_Master_Log_Pos: 13456620
Relay_Log_Space: 36764898503
Until_Condition: None
Until_Log_File:
Until_Log_Pos: 0
Master_SSL_Allowed: No
Master_SSL_CA_File:
Master_SSL_CA_Path:
Master_SSL_Cert:
Master_SSL_Cipher:
Master_SSL_Key:
Seconds_Behind_Master: 249904
××××××××××××××××××××××××××××××××××××××××××××××××××××××××××
```
5. 测试主从是否同步

由于slave是从master获取二进制日志来同步数据的，因此在slave上不应该有任何非查询类的sql语句执行，如果slave执行了sql语句操作了某个数据，当master操作这个数据时，salve通过二进制日志重演去操作时发生报错，会导致主从断开。如果主从断开我们就需要重新change master来建立主从。因此切记在单向主从同步的环境中，不可操作slave去执行非查询类操作。

我们在master上创建一个xiemx数据库，查看下slave是否会自动创建出来。
```
mysql> create database  xiemx；show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| test               |
| xiemx              |
+--------------------+
4 rows in set (0.00 sec)
```
如果如果在slave上show databases；结果和上面相同的则说明主从同步设置成功。