---
layout: post
title: twemproxy构建redis/memcache集群
author: xiemx
comments: true
date: 2017-02-09 05:02:12
tags: [ memcached, redis, twemproxy, cluster ]
categories:
    - memcached
    - redis
# permalink: /2017/02/09/twemproxy
---
twemproxy构建memcache集群，也可以用来构建reids集群。方法类似只是配置文件不同。

官网描述：
	`A fast, light-weight proxy for memcached and redis`

### 安装配置
```shell
$ apt-get install libtool automake -y
$ git clone https://github.com/twitter/twemproxy.git
$ cd twemproxy
$ autoreconf -fvi
$ ./configure
$ make
$ src/nutcracker -h


root@vagrant-ubuntu-trusty-64:~# nutcracker -h
This is nutcracker-0.4.1

Usage: nutcracker [-?hVdDt] [-v verbosity level] [-o output file]
                  [-c conf file] [-s stats port] [-a stats addr]
                  [-i stats interval] [-p pid file] [-m mbuf size]

Options:
  -h, --help             : this help
  -V, --version          : show version and exit
  -t, --test-conf        : test configuration for syntax errors and exit
  -d, --daemonize        : run as a daemon
  -D, --describe-stats   : print stats description and exit
  -v, --verbose=N        : set logging level (default: 5, min: 0, max: 11)
  -o, --output=S         : set logging file (default: stderr)
  -c, --conf-file=S      : set configuration file (default: conf/nutcracker.yml)
  -s, --stats-port=N     : set stats monitoring port (default: 22222)
  -a, --stats-addr=S     : set stats monitoring ip (default: 0.0.0.0)
  -i, --stats-interval=N : set stats aggregation interval in msec (default: 30000 msec)
  -p, --pid-file=S       : set pid file (default: off)
  -m, --mbuf-size=N      : set size of mbuf chunk in bytes (default: 16384 bytes)
```

  
### 创建集群
```shell
#用tweaproxy启动代理一组memcached／redis非常简单，只需要指定配置文件即可，官方也提供了范本在仓库的conf/下
vagrant@memcache-server:~$ cat nutcracker.yml
memcache:
  listen: 127.0.0.1:11211
  hash: fnv1a_64
  distribution: ketama
  auto_eject_hosts: true
  server_retry_timeout: 2000
  server_failure_limit: 1
  servers:
   - 127.0.0.1:11221:1
   - 127.0.0.1:11222:1
   - 127.0.0.1:11223:1

#启动后端的memcached
vagrant@memcache-server:~$ ps -ef | grep memcache
vagrant  17934     1  0 09:07 ?        00:00:00 memcached -d -p 11221
vagrant  17942     1  0 09:07 ?        00:00:00 memcached -d -p 11222
vagrant  17950     1  0 09:07 ?        00:00:00 memcached -d -p 11223
vagrant  18295 18083  0 09:20 pts/0    00:00:00 grep --color=auto memcache

vagrant@memcache-server:~$ nutcracker -c nutcracker.yml -d

vagrant@memcache-server:~$ telnet localhost 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set test 0 0 1
1
STORED
get test
VALUE test 0 1
1
END
```