---
layout: post
title: zabbix监控memcache
published: true
author: xiemx
comments: true
date: 2016-12-19 05:12:02
tags: [ zabbix, monitor, memcache ]
categories:
    - zabbix
#permalink: '/2016/12/19/zabbix-monitor-memcache'
---
### script

```shell
cat memcached.sh

#!/bin/bash
port=$1
mem_conn="/bin/nc 127.0.0.1 $port"

case $2 in 
  conn)
    conn=`echo -e "stats\nquit"|$mem_conn|grep curr_connections | awk '{print $3}' `
    echo $conn
    ;;
  bytes)
    bytes=`echo -e "stats\nquit"|$mem_conn|grep bytes|awk '{print $3}'`
    echo `echo $bytes |tr -d " "`
    ;;
  cmd_get)
    cmd_get=`echo -e "stats\nquit"|$mem_conn|grep cmd_get| awk '{print $3}' `
    echo $cmd_get
    ;;
  cmd_set)
    cmd_set=`echo -e "stats\nquit"|$mem_conn|grep cmd_set| awk '{print $3}' `
    echo $cmd_set
    ;;
  get_hits)
    get_hits=`echo -e "stats\nquit"|$mem_conn|grep get_hits| awk '{print $3}' `
    echo $get_hits
    ;;
  read_qps_sec)
    count1=`echo -e "stats\nquit"|$mem_conn|grep cmd_get| awk '{print $3}'|tr -d '\r' `
    sleep 1
    count2=`echo -e "stats\nquit"|$mem_conn|grep cmd_get| awk '{print $3}'|tr -d '\r' `
    count=` expr $count2 - $count1`
    echo $count
    ;;
  write_qps_sec)
    count1=`echo -e "stats\nquit"|$mem_conn|grep cmd_set| awk '{print $3}' `
    sleep 1
    count2=`echo -e "stats\nquit"|$mem_conn|grep cmd_set| awk '{print $3}' `
    count=`echo $count2 $count1|awk '{printf($1-$2)}'`
    echo $count
    ;;
  
  hit_target)
    cmd_get=`echo -e "stats\nquit"|$mem_conn|grep cmd_get| awk '{print $3}' `
    get_hits=`echo -e "stats\nquit"|$mem_conn|grep get_hits| awk '{print $3}' `
    hit_target=`echo $get_hits $cmd_get|awk '{printf($1*100/$2)}'`
    echo $hit_target
    ;;
  *)
    echo "please input conn|bytes|cmd_get|cmd_set|get_hits|read_qps_sec|write_qps_sec|hit_target"
    ;;
esac
```

### 配置文件放到zabbix的conf.d/目录下，

```shell
cat zabbix_memcache.conf
UserParameter=memcache[*],/opt/script/zabbix/memcached.sh $1 $2
```