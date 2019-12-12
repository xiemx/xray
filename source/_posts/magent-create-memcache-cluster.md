---
layout: post
title: magent构建memcache集群
published: true
author: xiemx
comments: true
date: 2017-02-09 07:02:18
tags: [ memcached, database, cluster ]
categories:
    - memcached
---
magent功能相当于一个proxy调度器，通过一定的算法将数据存到不同后端。magent只是一个调度器，不处理数据同步问题。（另外一个由twitter开源的twemproxy，功能作用类似。可参考：[twemproxy安装配置](/2017/02/09/2017-02-09-twemproxu/)）

部署：
  
memcached部署自行百度。
  
magent参考：[repcache＋magent构建高可用memcache](/2017/02/09/2017-02-09-repcache-magent/)

1. 启动3个Memcached进程,端口11221、11222、11223.
```shell
# memcached -u xiemx -d -p 11221
# memcached -u xiemx -d -p 11222
# memcached -u xiemx -d -p 11223

```
2. 启动magent进程，端口11211，11221、11222为master，11223为backup；
```shell
magent -p 12000 -s 127.0.0.1:11221 -s127.0.0.1:11222 -b 127.0.0.1:11223
```

3. telnet 11211的magent，set key1和set key2，根据算法，key1/key2将被分配到11221/11222和11213端口的Memcached；
```shell
[root@memcache ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
stats
memcached agent v0.4
matrix 1 -> 127.0.0.1:11221, pool size 0
matrix 2 -> 127.0.0.1:11222, pool size 0
END
set key1 0 0 1
1
STORED
set key2 0 0 1
2
STORED
quit
Connection closed by foreign host.

[root@memcache ~]# telnet 127.0.0.1 11221
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
END
get key2
VALUE key2 0 1
2
END
quit
Connection closed by foreign host.

[root@memcache ~]# telnet 127.0.0.1 11222
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
VALUE key1 0 1
1
END
get key2
END
quit
Connection closed by foreign host.

[root@memcache ~]# telnet 127.0.0.1 11223
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
VALUE key1 0 1
1
END
get key2
VALUE key2 0 1
2
END
quit
Connection closed by foreign host.
```
4. 模拟11211、11212端口的Memcached挂掉  
```shell
[root@memcache ~]# ps -ef | grep memcached
root       6589    1   0 01:25 ?        00:00:00 memcached -u xiemx -d -p 11221
root       6591    1   0 01:25 ?        00:00:00 memcached -u xiemx -d -p 11222
root       6593    1   0 01:25 ?        00:00:00 memcached -u xiemx -d -p 11223
root       6609   6509   001:44 pts/0     00:00:00 grep memcached
[root@memcache ~]# kill -9 6589
[root@memcache ~]# kill -9 6591
[root@memcache ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1 
VALUE key1 0 1
1
END
get key2
VALUE key2 0 1
2
END
quit
Connection closed by foreign host.
```

5. 模拟11211、11212端口的Memcached重启复活(恢复后的memcache是没有数据的，需要逻辑处理启动后导入数据)
```shell
[root@memcache ~]# memcached -m 1 -u root -d -l 127.0.0.1 -p 11221
[root@memcache ~]# memcached -m 1 -u root -d -l 127.0.0.1 -p 11222
[root@memcache ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
END
get key2
END
quit
Connection closed by foreign host.
```