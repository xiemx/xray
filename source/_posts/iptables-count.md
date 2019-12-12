---
layout: post
title: iptables count计数
published: true
author: xiemx
comments: true
date: 2017-04-11 05:04:23
tags: [ linux, iptables ]
categories:
    - linux
---
```
两个参数：
  --verbose	-v		verbose mode
  --zero    -Z [chain [rulenum]]  Zero counters in chain or all chains


p-hsg-cache-6% sudo iptables -nL -v -t nat
Chain PREROUTING (policy ACCEPT 50741 packets, 2370K bytes)
 pkts bytes target     prot opt in     out     source               destination
  436 22672 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6379 to:192.168.10.82:6379
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6380 to:192.168.10.81:6380
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6381 to:192.168.10.81:6381
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6382 to:192.168.10.81:6382
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6383 to:192.168.10.81:6383

Chain INPUT (policy ACCEPT 18988 packets, 1082K bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 10532 packets, 549K bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
10968  572K MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0

p-hsg-cache-6% sudo iptables -Z -t nat

p-hsg-cache-6% sudo iptables -nL -v -t nat
Chain PREROUTING (policy ACCEPT 12 packets, 624 bytes)
 pkts bytes target     prot opt in     out     source               destination
    1    52 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6379 to:192.168.10.82:6379
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6380 to:192.168.10.81:6380
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6381 to:192.168.10.81:6381
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6382 to:192.168.10.81:6382
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            192.168.10.226       tcp dpt:6383 to:192.168.10.81:6383

Chain INPUT (policy ACCEPT 12 packets, 624 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 6 packets, 312 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    7   364 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0
p-hsg-cache-6%

```