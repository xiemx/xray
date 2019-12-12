---
layout: post
title: Mysql-proxy代理程序
published: true
author: xiemx
comments: true
date: 2016-02-18 06:02:43
tags: [ mysql, mysql-proxy ]
categories:
    - mysql
---
1、安装Lua、glib、pkg-config、libevent、GCC、make等工具不同的环境下不同

2、安装mysql-proxy
```shell
tar xzf mysql-proxy-0.8.1.tar.gz
cd mysql-proxy-0.8.1
./configure  --prefix=/usr/local/mysql-proxy[root@node1 bin]# /usr/local/mysql-proxy/bin/mysql-proxy     --help-all
Usage:
mysql-proxy [OPTION...] - MySQL Proxy
Help Options:
-h, --help                                              Show help options
--help-all                                              Show all help options
--help-proxy                                            Show options for the proxy-module

proxy-module
  -P, --proxy-address=<host:port>                         listening address:port of the proxy-server (default: :4040)
  -r, --proxy-read-only-backend-addresses=<host:port>     address:port of the remote slave-server (default: not set)
  -b, --proxy-backend-addresses=<host:port>               address:port of the remote backend-servers (default: 127.0.0.1:3306)
--proxy-skip-profiling                                  disables profiling of queries (default: enabled)
--proxy-fix-bug-25371                                   fix bug #25371 (mysqld > 5.1.12) for older libmysql versions
-s, --proxy-lua-script=<file>                           filename of the lua script (default: not set)
--no-proxy                                              don't start the proxy-module (default: enabled)
--proxy-pool-no-change-user                             don't use CHANGE_USER to reset the connection coming from the pool (default: enabled)
--proxy-connect-timeout                                 connect timeout in seconds (default: 2.0 seconds)
--proxy-read-timeout                                    read timeout in seconds (default: 8 hours)
--proxy-write-timeout                                   write timeout in seconds (default: 8 hours)

Application Options:
-V, --version                                           Show version
--defaults-file=<file>                                  configuration file
--verbose-shutdown                                      Always log the exit code when shutting down
--daemon                                                Start in daemon-mode
--user=<user>                                           Run mysql-proxy as user
--basedir=<absolute path>                               Base directory to prepend to relative paths in the config
--pid-file=<file>                                       PID file in case we are started as daemon
--plugin-dir=<path>                                     path to the plugins
--plugins=<name>                                        plugins to load
--log-level=(error|warning|info|message|debug)          log all messages of level ... or higher
--log-file=<file>                                       log all messages in a file
--log-use-syslog                                        log all messages to syslog
--log-backtrace-on-crash                                try to invoke debugger on crash
--keepalive                                             try to restart the proxy if it crashed
--max-open-files                                        maximum number of open files (ulimit -n)
--event-threads                                         number of event-handling threads (default: 1)
--lua-path=<...>                                        set the LUA_PATH
--lua-cpath=<...>                                       set the LUA_CPATH
```