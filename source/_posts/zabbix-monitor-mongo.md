---
layout: post
title: zabbix监控mongo
published: true
author: xiemx
comments: true
date: 2016-12-19 05:12:27
tags: [ zabbix, mongo ]
categories:
    - zabbix
#permalink: '/2016/12/19/zabbix-monitor-mongo'
---
可配合zabbix自动发现，自动监控服务
### script

```shell
cat mongo.sh

#!/bin/bash

case $1 in 
#  use_memory)
#    used_memory=`echo "db.serverStatus().mem"|mongo admin|grep resident|awk -F':' '{print $2}'|tr -d " ,"`
#    echo $used_memory
#    ;;
#  
#  use_vmemory)
#    used_vmemory=`echo "db.serverStatus().mem"|mongo admin|grep virtual|awk -F':' '{print $2}'|tr -d " ,"`
#    echo $used_vmemory
#    ;;
#  
#  used_conn)
#    used_conn=`echo "db.serverStatus().connections"|mongo admin|grep current|awk -F':' '{print $2}'|tr -d ' ,'`
#    echo $used_conn
#    ;;
#  
#  available_conn)
#    available=`echo "db.serverStatus().connections"|mongo admin|grep available|awk -F':' '{print $2}'|tr -d ' ,'`
#    echo $availabe
#  ;;
  
  insert)
    insert=`mongostat -n1 | tail -n 1|awk '{print $1}'|tr -d ' *,'`
    echo $insert
    ;;
  
  query)
    query=`mongostat -n1 | tail -n 1|awk '{print $2}'|tr -d ' *,'`
    echo $query
    ;;
  
  update)
    update=`mongostat -n1 | tail -n 1|awk '{print $3}'|tr -d ' *,'`
    echo $update
    ;;
  
  delete)
    delete=`mongostat -n1 | tail -n 1|awk '{print $4}'|tr -d ' *,'`
    echo $delete
    ;;
  getmore)
    getmore=`mongostat -n1 | tail -n 1|awk '{print $5}'|tr -d ' *,'`  
    echo $getmore
    ;;
  command)
    command=`mongostat -n1 | tail -n 1|awk '{print $6}'|awk -F'|' '{print $1}'|tr -d ' *,'`
    echo $command
    ;;
  vsize)
    vsize=`mongostat -n1 | tail -n 1|awk '{print $10}'|tr -d ' *,G'`
    echo $vsize
    ;;
  res)
    res=`mongostat -n1 | tail -n 1|awk '{print $11}'|tr -d ' *,G'`
    echo $res
    ;;
  qr)
    qr=`mongostat -n1 | tail -n 1|awk '{print $12}'|awk -F'|' '{print $1}'|tr -d ' *,'`
    echo $qr
    ;;
  qw)
    qw=`mongostat -n1 | tail -n 1|awk '{print $12}'|awk -F'|' '{print $2}'|tr -d ' *,'`
    echo $qw
    ;;
  ar)
    ar=`mongostat -n1 | tail -n 1|awk '{print $13}'|awk -F'|' '{print $1}'|tr -d ' *,'`
    echo $ar
    ;;
  aw)
    aw=`mongostat -n1 | tail -n 1|awk '{print $13}'|awk -F'|' '{print $2}'|tr -d ' *,'`
    echo $aw
    ;;
  conn)
    conn=`mongostat -n1 | tail -n 1|awk '{print $16}'|tr -d ' *,'`
    echo $conn
    ;;

  *)
    echo "please input  insert|query|update|delete"
    ;;
esac
```


### conf配置文件

```shell
cat zabbix_mongo.conf
UserParameter=mongo[*],/opt/script/zabbix/mongo.sh $1
```