---
layout: post
title: zabbix监控redis
published: true
author: xiemx
comments: true
date: 2016-12-19 05:12:43
tags: [ zabbix, scripts ]
categories:
    - zabbix
#permalink: '/2016/12/19/zabbix-monitor-redis'
---
可配合zabbix自动发现，来完成自动监控

### scripts

```shell
cat redis.sh
#!/bin/bash
redis_conn="/usr/local/bin/redis-cli"
port=$1

case $2 in 
  "used_memory")
    used_memory=`$redis_conn -p $port info | grep used_memory:|awk -F':' '{print $2}'`
    echo $used_memory
    ;;
  "ops_sec")
    ops=`$redis_conn -p $port info|grep instantaneous_ops_per_sec:|awk -F':' '{print $2}'`
    echo $ops
    ;;
  "connected_clients")
    connected_clients=`$redis_conn -p $port info|grep connected_clients: | awk -F ':' '{print $2}'`
    echo $connected_clients
    ;;
  "blocked_clients")
    blocked_clients=`$redis_conn -p $port info|grep blocked_clients:|awk -F':' '{print $2}'`
    echo $blocked_clients
    ;; 
  *)
    echo "please input used_memory|ops_sec|connected_clients|blocked_clients"
    ;; 
esac
```

### conf文件

```yaml
cat zabbix_redis.conf

UserParameter=redis[*],/opt/script/zabbix/redis.sh $1 $2
```
