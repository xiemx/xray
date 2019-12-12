---
layout: post
title: mysql锁表解决
published: true
author: xiemx
comments: true
date: 2016-08-02 02:08:56
tags: [ mysql, debug, database ]
categories:
    - mysql
# permalink: '/2016/08/02/mysql%e9%94%81%e8%a1%a8%e8%a7%a3%e5%86%b3'
---
1、查询是否锁表
```sql
show OPEN TABLES where In_use > 0;
```
2、查询进程
```
show processlist 查询到相对应的ID
```
3、杀死进程
```
kill id
```
other:
```
查看正在锁的事务
SELECT * FROMINFORMATION_SCHEMA.INNODB_LOCKS; 

查看等待锁的事务
SELECT * FROMINFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```