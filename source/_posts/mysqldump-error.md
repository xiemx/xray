---
layout: post
title: Mysqldump error
published: true
author: xiemx
comments: true
date: 2019-05-06 11:05:56
tags: [ mysql, debug ,database]
categories:
    - mysql
# permalink: /2019/05/06/mysqldump-error
---
### 现象
```
[root@FCHK-instance ~]# mysqldump --host rm-xxxxxxxxxxx.mysql.rds.aliyuncs.com -u xxxx -p --databases visa > hk.sql
Enter password:
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events.
mysqldump: Couldn't execute 'SELECT COLUMN_NAME,                       JSON_EXTRACT(HISTOGRAM, '$."number-of-buckets-specified"')                FROM information_schema.COLUMN_STATISTICS                WHERE SCHEMA_NAME = 'visa' AND TABLE_NAME = 'admin';': Unknown table 'column_statistics' in information_schema (1109
```
### 分析
```

[root@FCHK-instance ~]# mysql --version
mysql  Ver 8.0.11 for Linux on x86_64 (MySQL Community Server - GPL)

可能是由于mysqldump 8中默认启用（COLUMN_STATISTICS）

官方文档解释
Mysql 8.0 The INFORMATION_SCHEMA COLUMN_STATISTICS Table
https://dev.mysql.com/doc/refman/8.0/en/column-statistics-table.html
```
### 解决
```
[root@FCHK-instance ~]# mysqldump --host rm-xxxxxxxxxx2.mysql.rds.aliyuncs.com -u xxxx -p --databases visa --column-statistics=0 > hk.sql
Enter password:
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events.
```