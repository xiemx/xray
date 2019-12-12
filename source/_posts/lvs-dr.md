---
layout: post
title: LVS-DR集群构建
published: true
author: xiemx
comments: true
date: 2016-02-18 04:02:11
tags: [ lvs, cluster ]
categories:
    - lvs
---
设置端口标记的规则
```shell
#!/bin/bash
VIP=$1
IPTABLES=/sbin/iptables
$IPTABLES -t mangle -A PREROUTING -p tcp -d $VIP --dport 80 -j MARK --set-mark 100
$IPTABLES -t mangle -A PREROUTING -p tcp -d $VIP --dport 443 -j MARK --set-mark 100
```
ARP防火墙设置
```shell
#!/bin/bash
VIP=192.168.0.100
RIP=192.168.0.x
DGW=172.25.0.254
DGWMAC=52:54:00:00:00:fe
arptables -F
arptables -A IN -d $VIP -j DROP
arptables -A OUT -s $VIP -j mangle --mangle-ip-s $RIP
/sbin/ifconfig eth0:1 $VIP broadcast $VIP netmask 255.255.255.0 up
arp -s $DGW  $DGWMAC
/sbin/route add default gw $DGW
```
 检测脚本
```shell
#!/bin/bash
/usr/bin/links -dump 1 $1 >/dev/null 2>&1
if [ 0 -eq $? ] ; then
    echo ok
else
    echo fail
fi
```
主配置文件
```conf
[root@lvs-f30 ~]# cat /etc/sysconfig/ha/lvs.cf
serial_no = 25
primary = 172.25.30.14
primary_private = 192.168.122.246
service = lvs
backup_active = 1
backup = 172.25.30.15
backup_private = 192.168.122.247
heartbeat = 1
heartbeat_port = 539
keepalive = 6
deadtime = 18
network = direct
debug_level = NONE
monitor_links = 0
syncdaemon = 0
syncd_iface = eth0

virtual http {
     active = 1
     address = 172.25.30.100 eth0:1
     vip_nmask = 255.255.255.0
     fwmark = 100
     port = 80
     send = "GET / HTTP/1.0\r\n\r\n"
     expect = "ok"
     use_regex = 0
     send_program = "/bin/testlink %h"
     load_monitor = none
     scheduler = wlc
     protocol = tcp
     timeout = 6
     reentry = 15
     quiesce_server = 0

     server node1 {
         address = 192.168.122.224
         active = 1
         port = 80
         weight = 1
     }

     server node2 {
         address = 192.168.122.245
         active = 1
         port = 80
         weight = 1
     }
}

virtual https {
     active = 1
     address = 172.25.30.100 eth0:1
     vip_nmask = 255.255.255.0
     fwmark = 100
     port = 443
     send = "GET / HTTP/1.0\r\n\r\n"
     expect = "HTTP"
     use_regex = 0
     load_monitor = none
     scheduler = wlc
     protocol = tcp
     timeout = 6
     reentry = 15
     quiesce_server = 0

     server node1 {
         address = 192.168.122.224
         active = 1
         port = 443
         weight = 1
     }

     server node2 {
         address = 192.168.122.245
         active = 1
         port = 443
         weight = 1
     }
}
```