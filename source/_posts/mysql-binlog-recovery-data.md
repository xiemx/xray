---
layout: post
title: mysql binlog恢复数据
published: true
author: xiemx
comments: true
date: 2016-01-30 11:01:53
tags: [ mysql, binlog ]
categories:
    - mysql
---
```shell
[root@localhost mysql]# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
log-bin=/var/log/mysql/binlog   开启二进制日志
socket=/var/lib/mysql/mysql.sock
user=mysql
log-slow-queries=/var/log/mysql/slowlog
symbolic-links=0
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[root@localhost mysql]# cd /var/log/mysql
[root@localhost mysql]# ls
binlog.000001  binlog.000002  binlog.000003  binlog.index             二进制生成的文件
```
查看数据库的日志相关设置：
```sql
mysql> SHOW  GLOBAL VARIABLES LIKE '%log%';
+-----------------------------------------+----------------------------+
| Variable_name                           | Value                      |
+-----------------------------------------+----------------------------+
| back_log                                | 50                         |
| binlog_cache_size                       | 32768                      |
| binlog_direct_non_transactional_updates | OFF                        |
| binlog_format                           | STATEMENT                  |
| expire_logs_days                        | 0                          |
| general_log                             | OFF                        |
| general_log_file                        | /var/run/mysqld/mysqld.log |
| innodb_flush_log_at_trx_commit          | 1                          |
| innodb_locks_unsafe_for_binlog          | OFF                        |
| innodb_log_buffer_size                  | 1048576                    |
| innodb_log_file_size                    | 5242880                    |
| innodb_log_files_in_group               | 2                          |
| innodb_log_group_home_dir               | ./                         |
| innodb_mirrored_log_groups              | 1                          |
| log                                     | OFF                        |
| log_bin                                 | ON                         |    查看二进制日志是否开启
| log_bin_trust_function_creators         | OFF                        |
| log_bin_trust_routine_creators          | OFF                        |
| log_error                               | /var/log/mysqld.log        |   错误日志
| log_output                              | FILE                       |
| log_queries_not_using_indexes           | OFF                        |
| log_slave_updates                       | OFF                        |
| log_slow_queries                        | ON                         |
| log_warnings                            | 1                          |
| max_binlog_cache_size                   | 18446744073709547520       |
| max_binlog_size                         | 1073741824                 |
| max_relay_log_size                      | 0                          |
| relay_log                               |                            |   中继日志
| relay_log_index                         |                            |
| relay_log_info_file                     | relay-log.info             |
| relay_log_purge                         | ON                         |
| relay_log_space_limit                   | 0                          |
| slow_query_log                          | ON                         |   慢查询日志
| slow_query_log_file                     | /var/log/mysql/slowlog     |
| sql_log_bin                             | ON                         |
| sql_log_off                             | OFF                        |
| sql_log_update                          | ON                         |
| sync_binlog                             | 0                          |
+-----------------------------------------+----------------------------+
38 rows in set (0.00 sec)

```

查看二进制日志文件：二进制文件不是文本文档无法使用cat等于都文本的命令查看，需使用mysqlbinlog命令查看。
```sql
[root@localhost mysql]# mysqlbinlog binlog.000003
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#160110 16:41:49 server id 1  end_log_pos 106     Start: binlog v 4, server v 5.1.73-log created 160110 16:41:49 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
TRmSVg8BAAAAZgAAAGoAAAABAAQANS4xLjczLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAABNGZJWEzgNAAgAEgAEBAQEEgAAUwAEGggAAAAICAgC
'/*!*/;
# at 106                 position号标记
#160110 17:26:15 server id 1  end_log_pos 191     Query    thread_id=3    exec_time=0    error_code=0    时间标记160110表示2016-01-10
SET TIMESTAMP=1452417975/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=1, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=0/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C latin1 *//*!*/;
SET @@session.character_set_client=8,@@session.collation_connection=8,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
create database xiemx     对数据库的操作记录
/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
```

二进制恢复命令：
```
[root @server1 mysql ] #mysqlbinlog --start-date="2015-06-18 9:55:00" --stop-date="2015-06-18 10:05:00" /var/log/mysql/binlog.000006 | mysql -uroot -p

从二进制日志binlog.000006的2015-06-18 9:55:00这个时间到2015-06-18 10:05:00之间的操作

[root @server1 mysql ] #mysqlbinlog --stop-position="368312" /var/log/mysql/binlog.000006 | mysql -u root -p  从二进制日志binlog.000006的开头恢复到368312这个position标记结束

[root @server1 mysql ] #mysqlbinlog --start-position="368315" /var/log/mysql/bin.000006 | mysql -u root -p     从二进制日志binlog.000006的368315这个position标记恢复到日志结束
```
在使用二进制恢复数据库时可以使用position标记和时间标记，假设我们在数据库操作中误执行了错误的操作我们可以用position和时间标记来跳 过错误的操作不执行，恢复出数据库。二进制日志恢复数据库是基于某个完整备份的基础上。因此在备份数据库时我们可以使用flush log命令来刷新二进制文件，记录下文件名。这样下次数据库出现问题我们就可以用手上的二进制文件和备份来恢复数据。

 