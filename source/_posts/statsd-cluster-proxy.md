---
layout: post
title: statsd cluster proxy
published: true
author: xiemx
comments: true
date: 2017-09-08 05:09:37
tags: [ statsd, graphite, monitor, database]
categories:
    - statsd
    - graphite
# permalink: /2017/09/08/statsd-cluster-proxy
---
通过UDP proxy程序将前端的数据通过一定的hash算法将相同的metric发送的固定的后aggregation数据。proxy代理支持健康检测自动剔除／加入后端statsd。

1.配置
本例展示一个3节点的statsd后端且将聚合数据发送到standout方便观测。

```
node1
######config文件：
{
 port: 8127
, mgmt_port: 8227
, backends: [ "./backends/console" ]
}

启动:
xiemx➜  statsd : master ✘ :✹✭ ᐅ  node stats.js config8127.js
8 Sep 11:06:36 - [81604] reading config file: config8127.js
8 Sep 11:06:36 - server is up INFO
```
```
node2
######config文件：
{
 port: 8128
, mgmt_port: 8228
, backends: [ "./backends/console" ]
}

启动：
xiemx➜  statsd : master ✘ :✹✭ ᐅ  node stats.js config8128.js
8 Sep 11:07:09 - [81665] reading config file: config8128.js
8 Sep 11:07:09 - server is up INFO
```
```
node3
######config文件：
{
 port: 8129
, mgmt_port: 8229
, backends: [ "./backends/console" ]
}

启动：
xiemx➜  statsd : master ✘ :✹✭ ᐅ  node stats.js config8129.js
8 Sep 11:07:45 - [81723] reading config file: config8129.js
8 Sep 11:07:45 - server is up INFO
```
```
proxy
######config文件：
{
nodes: [
{host: '127.0.0.1', port: 8127, adminport: 8227},
{host: '127.0.0.1', port: 8128, adminport: 8228},
{host: '127.0.0.1', port: 8129, adminport: 8229}
],
server: './servers/udp',
host:  '0.0.0.0',
port: 8125,
mgmt_port: 8126,
forkCount: 0,
checkInterval: 1000,
cacheSize: 10000,
deleteIdleStats: true
}

启动：
xiemx➜  statsd : master ✘ :✹✭ ᐅ  node proxy.js proxyconfig.js
8 Sep 11:09:02 - [81938] reading config file: proxyconfig.js
8 Sep 11:09:02 - INFO: [81938] server is up
```

测试：
5个线程同时推送500个metric到代理查看分片情况

```
测试命令：
for i in $(seq 1 500);do echo "Ezbuy-$i:1|c" | nc -u -w0 127.0.0.1 8125;done
```

![img](/images/img_59b269634a216.png)

![img](/images/img_59b2697b89208.png)

节点自动剔除和加入：

![img](/images/img_59b2698d5658f.png)

