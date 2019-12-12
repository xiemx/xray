---
layout: post
title: keepalived配置多实例
published: true
author: xiemx
comments: true
date: 2016-08-24 04:08:05
tags: [ keepalived, cluster]
categories:
    - keepalived
---

  
安装keepalived

1、 下载 wget http://www.keepalived.org/software/keepalived-1.2.23.tar.gz
  
2、 解包 tar xf keepalived-1.2.23.tar.gz
  
3、 切换目录 cd keepalived-1.2.23
  
4、 配置 ./configure –prefix=/opt/keepalived 因为keepalived 运行在ipvs 之上，因此这两
  
个软件一定要安装在一个系统里面。如果configure 操作能正常进行，运行完毕后将有如下的汇总输出：
```
Keepalived configuration
------------------------
Keepalived version : 1.2.23
Compiler : gcc
Compiler flags : -g -O2
Extra Lib : -lpopt -lssl -lcrypto
Use IPVS Framework : Yes
IPVS sync daemon support : Yes
Use VRRP Framework : Yes
Use LinkWatch : No
Use Debug flags : No
```
5、 编译和安装 
make&&make install
6、 环境配置
```
cd /opt/keepalived/
mkdir /etc/keepalived/
cp sbin/keepalived /usr/sbin/
cp etc/keepalived/keepalived.conf /etc/keepalived/
cp etc/sysconfig/keepalived /etc/sysconfig/
cp etc/rc.d/init.d/keepalived /etc/init.d/
```
7、启动keepalived查看是否正常
```
[root@cluster-node1 keepalived]# /etc/init.d/keepalived start
Starting keepalived (via systemctl):  [  OK  ]
[root@cluster-node1 keepalived]# ps -ef | grep keepalived
root       4662      1  0 01:22 ?        00:00:00 keepalived -D
root       4664   4662  0 01:22 ?        00:00:00 keepalived -D
root       4665   4662  0 01:22 ?        00:00:00 keepalived -D
root      16435   2705  0 01:37 pts/0    00:00:00 grep --color=auto keepalived
```
8、修改配置文件
```
[root@cluster-node1 keepalived]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id node1
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface eno16777736 
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.1/24
    }
}
vrrp_instance VI_2 {
    state BACKUP
    interface eno16777736 
    virtual_router_id 52
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.1.1/24
    }
}
```
```
[root@cluster-node2 keepalived]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id node2
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno16777736
    virtual_router_id 51
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.1/24
    }
}
vrrp_instance VI_2 {
    state MASTER
    interface eno16777736
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.1.1/24
    }
}
```
9、重启keepalived，查看vip是否正常绑定
```
[root@cluster-node1 keepalived]# ip add | egrep  "0.1|1.1"
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 10.0.0.1/24 scope global eno16777736
[root@cluster-node2 keepalived]# ip add | egrep  "0.1|1.1"
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 10.0.1.1/24 scope global eno16777736
```
