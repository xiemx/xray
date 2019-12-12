---
layout: post
title: mysql master-master
published: true
author: xiemx
comments: true
date: 2016-01-31 12:01:23
tags: [ mysql ]
categories:
    - mysql
---

原理同单向主从相同，都是salve获取master的二进制日志来重演数据，在互为主从的架构中，servera既是master又是slave，通用的serverb也是master和slave，servera回去同步serverb的数据，serverb也会去同步servera的操作。已达到数据的一致性。

由原理可以得知：

* servera需要开启二进制日志授权serverb来同步，servera需要change master指向serverb

* serverb需要开启二进制日志授权servera来同步，serverb需要change master指向servera，另外serverb的数据是通过master来同步到中继日志中来重演生成的，因此需要在开启将中继日志写入二进制日志功能`log_slave_update=1`

需要注意的是，在设置双向主从时，我们是先设置一条单向主从(servera-serverb)，在设置另一条单向主从(serverb-servera)。我们在设置完第一条单向主从(servera-serverb)成功时，slave(serverb)上是不能执行任何非查询语句的，因此第二条单向主从(serverb-servera)在serverb上设置servera的同步账户时grant授权语句需要通过(servera-serverb)这条单向主从的master(servera)来传递给salve(serverb)。

#### 操作步骤

1. 设置server-id，开启二进制日志，serverb需要开启中继日志写入二进制日志功能
```shell
[root@localhost mysql]# cat /etc/my.cnf
[mysqld]
server-id=1            master/slave数字不可重复
datadir=/var/lib/mysql
log-bin=/var/log/mysql/binlog
socket=/var/lib/mysql/mysql.sock
log_slave_update=1              serverb上添加此项
log-slow-queries=/var/log/mysql/slowlog
symbolic-links=0
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```
2. servera创建同步用户，并授权
```sql
mysql>grant   replication  slave on *.*  to xiemx1@‘172.25.30.%’ identified by '123123';flush privileges;
```
3. serverb指定master信息
```
mysql>change master to master_host='172.25.30.10',master_user='xiemx1',master_password='123123',master_log_file="binlog.000003"，master_log_pos=245;

mysql>slave start；

mysql>show slave status\G;查看io和sql是否yes，在测试主从是否生效，在保障主从生效的情况下执行下列操作。
```
4. servera上授权第二条单向的同步账户
```
mysql>grant   replication  slave on *.*  to xiemx2@‘172.25.30.%’ identified by '123123';flush privileges;
```
5. servera指定master信息
```
mysql>change master to master_host='172.25.30.11',master_user='xiemx2',master_password='123123',master_log_file="binlog.000001"，master_log_pos=245;

mysql>slave start；

mysql>show slave status\G;查看io和sql是否为yes
```
6. 在servera上创建一个库看serverb是否会自动生成，在serverb上创建库看servera是否会自动生成。
如果都没有问题的话说明设置成功。
`show master status\G;` 可以查看当前的二进制日志文件和position号