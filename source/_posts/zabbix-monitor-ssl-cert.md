---
layout: post
title: zabbix 监控ssl证书过期时间
published: true
author: xiemx
comments: true
date: 2016-12-05 10:12:45
tags: [ ]
categories:
    - zabbix
#permalink: '/2016/12/05/zabbix-monitor-ssl-cert'
---
脚本是参考网上修改过的版本，注意要指定servername参数，以免抓取到其它站点的证书。

```shell
mingxu.xie@t-slq-ops-1:/opt/script/zabbix# cat check_ssl_cert.sh 
#/bin/bash
host=1
port=2
end_date=openssl s_client -host $host -port $port -servername $host -showcerts &lt;/dev/null 2>/dev/null|sed -n '/BEGIN CERTIFICATE/,/END CERT/p' |openssl x509 -text 2>/dev/null |sed -n 's/ *Not After : *//p'
if [ -n "end_date" ];then
    end_date_seconds=`date '+%s' --date "end_date"now_seconds=date '+%s'`
    echo "(end_date_seconds-now_seconds)/24/3600" | bc
fi
```