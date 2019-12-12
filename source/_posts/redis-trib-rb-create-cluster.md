---
layout: post
title: redis-trib.rb构建redis集群
published: true
author: xiemx
comments: true
date: 2017-02-27 04:02:11
tags: [ redis, cluster ]
categories:
    - redis
# permalink: '/2017/02/27/redis-trib-rb%e6%9e%84%e5%bb%baredis%e9%9b%86%e7%be%a4'
---
redis-cluster

使用官方的redis-trib.rb构建redis集群,要让redis cluster集群正常工作需要有三个主节点，因此我们在三台机器上部署6个节点，每台机器一个master一个slave。

环境：
p-hsg-redis-1  192.168.10.81   7000/7001
p-hsg-redis-2	 192.168.10.82   7000/7001
p-hsg-redis-3	 192.168.10.83   7000/7001


部署流程：

1.安装redis－server

我这里已经有saltstack的自动化构建脚本，因此使用salt安装软件。具体redis安装方法不再赘述。
salt-call state.sls redis.cluster

2.修改配置文件
```shell
	root@p-hsg-redis-1:/etc/redis# cat /etc/redis/7000.conf
	protected-mode no
	daemonize yes
	port 7000
	cluster-enabled yes
	cluster-config-file nodes-7000.conf
	cluster-node-timeout 15000
	appendonly yes
	dir /var/lib/redis/7000
```
3.启动redis节点
```shell
	root@p-hsg-redis-3:/etc/redis# ps -ef | grep redis
	root     30431     1  0 15:12 ?        00:00:00 /usr/local/bin/redis-server *:7001 [cluster]
	root     30446     1  0 15:12 ?        00:00:00 /usr/local/bin/redis-server *:7000 [cluster]
	root     30450 11073  0 15:12 pts/1    00:00:00 grep --color=auto redis
```
4.创建集群
```shell
	redis-trib.rb create --replicas 1 192.168.10.81:7001 192.168.10.81:7000 192.168.10.82:7000 192.168.10.82:7001 192.168.10.83:7000 192.168.10.83:7001
	>>> Creating cluster
	>>> Performing hash slots allocation on 6 nodes...
	Using 3 masters:
	192.168.10.81:7001
	192.168.10.82:7000
	192.168.10.83:7000
	Adding replica 192.168.10.82:7001 to 192.168.10.81:7001
	Adding replica 192.168.10.81:7000 to 192.168.10.82:7000
	Adding replica 192.168.10.83:7001 to 192.168.10.83:7000
	M: 5fd250591aa474d6370e1df547959c5895716192 192.168.10.81:7001
	   slots:0-5460 (5461 slots) master
	S: ef8bf8a326894fee37a398d3f88de3120fc71ea5 192.168.10.81:7000
	   replicates dc62f19b8894db1816f7b92e3fe05b8244832d3e
	M: dc62f19b8894db1816f7b92e3fe05b8244832d3e 192.168.10.82:7000
	   slots:5461-10922 (5462 slots) master
	S: eb2c7cc3cb56bb5ee124d80e353a91331d092042 192.168.10.82:7001
	   replicates 5fd250591aa474d6370e1df547959c5895716192
	M: 1773d96052f573361a13b5e9015f947b340d45cd 192.168.10.83:7000
	   slots:10923-16383 (5461 slots) master
	S: 670f158355b93b4a7ac30d69dc3e1e8c4c484a9f 192.168.10.83:7001
	   replicates 1773d96052f573361a13b5e9015f947b340d45cd
	Can I set the above configuration? (type 'yes' to accept): yes
	>>> Nodes configuration updated
	>>> Assign a different config epoch to each node
	>>> Sending CLUSTER MEET messages to join the cluster
	Waiting for the cluster to join...
	>>> Performing Cluster Check (using node 192.168.10.81:7001)
	M: 5fd250591aa474d6370e1df547959c5895716192 192.168.10.81:7001
	   slots:0-5460 (5461 slots) master
	   1 additional replica(s)
	M: 1773d96052f573361a13b5e9015f947b340d45cd 192.168.10.83:7000
	   slots:10923-16383 (5461 slots) master
	   1 additional replica(s)
	S: ef8bf8a326894fee37a398d3f88de3120fc71ea5 192.168.10.81:7000
	   slots: (0 slots) slave
	   replicates dc62f19b8894db1816f7b92e3fe05b8244832d3e
	S: 670f158355b93b4a7ac30d69dc3e1e8c4c484a9f 192.168.10.83:7001
	   slots: (0 slots) slave
	   replicates 1773d96052f573361a13b5e9015f947b340d45cd
	M: dc62f19b8894db1816f7b92e3fe05b8244832d3e 192.168.10.82:7000
	   slots:5461-10922 (5462 slots) master
	   1 additional replica(s)
	S: eb2c7cc3cb56bb5ee124d80e353a91331d092042 192.168.10.82:7001
	   slots: (0 slots) slave
	   replicates 5fd250591aa474d6370e1df547959c5895716192
	[OK] All nodes agree about slots configuration.
	>>> Check for open slots...
	>>> Check slots coverage...
	[OK] All 16384 slots covered.
	```