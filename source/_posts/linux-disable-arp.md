---
layout: post
title: arp_ignore和arp_announce参数说明
published: true
author: xiemx
comments: true
date: 2016-08-11 04:08:15
tags: [ linux, arp ]
categories:
    - linux
---
lvsDR模式中需要对real server进行arp抑制，我们通常会在sysctl.conf文件中修改如下内核参数：
```
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
arp_ignore:定义对目标地址为本地IP的ARP询问不同的应答模式0 

0 - (默认值): 回应任何网络接口上对任何本地IP地址的arp查询请求 
1 - 只回答目标IP地址是来访网络接口本地地址的ARP查询请求 
2 -只回答目标IP地址是来访网络接口本地地址的ARP查询请求,且来访IP必须在该网络接口的子网段内 
3 - 不回应该网络界面的arp请求，而只对设置的唯一和连接地址做出回应 
4-7 - 保留未使用 
8 -不回应所有（本地地址）的arp查询

arp_announce:对网络接口上，本地IP地址的发出的，ARP回应，作出相应级别的限制: 确定不同程度的限制,宣布对来自本地源IP地址发出Arp请求的接口 

0 - (默认) 在任意网络接口（eth0,eth1，lo）上的任何本地地址 
1 -尽量避免不在该网络接口子网段的本地地址做出arp回应. 当发起ARP请求的源IP地址是被设置应该经由路由达到此网络接口的时候很有用.此时会检查来访IP是否为所有接口上的子网段内ip之一.如果改来访IP不属于各个网络接口上的子网段内,那么将采用级别2的方式来进行处理. 
2 - 对查询目标使用最适当的本地地址.在此模式下将忽略这个IP数据包的源地址并尝试选择与能与该地址通信的本地地址.首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址. 如果没有合适的地址被发现,将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送.
```
