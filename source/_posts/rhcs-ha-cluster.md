---
layout: post
title: rhcs搭建HA集群
published: true
author: xiemx
comments: true
date: 2016-02-01 12:02:16
tags: [ rhcs, cluster ]
categories:
    - cluster
# permalink: '/2016/02/01/rhcs-ha-cluster'
---
YUM配置HA集群(图形化)

RHEL和Centos的光盘中自带有红帽的conga套件。将安装源指向光盘即可yum来安装，也可以将luci和ricci拷贝出来直接通过"yum localinstall luci ricci"来安装。其中luci是web界面的图形化配置工具，ricci为同步配置文件的工具。ricci运行需要用到ricci账户权限，安装ricci时设定下ricci密码。

安装设置步骤

1.所有节点进行环境初始化

* 时间日期（date）
* elinux
* 防火墙（iptables）
* 主机名（/etc/hosts和/etc/sysconfig/network）
* 网卡网络(stop NetworkManager)
* yum源

2.在所有节点上安装ricci，在其中一台节点上安装luci，并配置ricci密码。
```shell
[root@localhost ~]# yum install luci ricci -y
[root@localhost ~]# yum install ricci -y
[root@localhost ~]# echo 123123123 | passwd --stdin ricci
```
3.启动luci进入web配置界面，可使用系统root账户登录luci
```shell
[root@localhost ~]# /etc/init.d/luci start
Start luci...                       [ OK ]
Point your web browser to https://localhost.localdomain:8084 (or equivalent) to access luci

[![ha](/images/ha.png)](http://www.xiemx.com/wp-content/uploads/2016/01/ha.png)

[![hainstall](/images/hainstall.png)](http://www.xiemx.com/wp-content/uploads/2016/02/hainstall.png)

在双机集群的基础上添加一个节点进去实现多机集群，这里使用非图形化安装
安装 High Availability 组包

[root@node3 ~]# yum groupinstall "High Availability"
设置ricci用户密码:
[root@node1 ~]# echo uplooking | passwd --stdin ricci
Changing password for user ricci.
passwd: all authentication tokens updated successfully.
```
修改配置文件/etc/cluster/cluster.conf并同步到所有结点上
```xml
<?xml version="1.0" encoding="utf-8"?>

<cluster config_version="12" name="f30Cluster"> 
  <clusternodes> 
    <clusternode name="node1.xiemx.com" nodeid="1"> 
      <fence> 
        <method name="FenceMethod"> 
          <device domain="node1" name="Fence1"/> 
        </method> 
      </fence> 
    </clusternode>  
    <clusternode name="node2.xiemx.com" nodeid="2"> 
      <fence> 
        <method name="FenceMethod"> 
          <device domain="node2" name="Fence2"/> 
        </method> 
      </fence> 
    </clusternode>  
    <clusternode name="node3.xiemx.com" nodeid="3"> 
      <fence> 
        <method name="FenceMethod"> 
          <device domain="node3" name="Fence3"/> 
        </method> 
      </fence> 
    </clusternode> 
  </clusternodes>  
  <cman expected_votes="1"/>  
  <fencedevices> 
    <fencedevice agent="fence_xvm" name="Fence1"/>  
    <fencedevice agent="fence_xvm" name="Fence2"/>  
    <fencedevice agent="fence_xvm" name="Fence3"/> 
  </fencedevices>  
  <rm> 
    <failoverdomains> 
      <failoverdomain name="Domain1" restricted="1"> 
        <failoverdomainnode name="node1.xiemx.com"/>  
        <failoverdomainnode name="node2.xiemx.com"/>  
        <failoverdomainnode name="node3.xiemx.com"/> 
      </failoverdomain> 
    </failoverdomains>  
    <resources> 
      <ip address="172.25.30.100/24" sleeptime="10"/>  
      <script file="/etc/init.d/httpd" name="httpd"/> 
    </resources>  
    <service domain="Domain1" name="httpd" recovery="restart"> 
      <ip ref="172.25.30.100/24"/>  
      <script ref="httpd"/> 
    </service> 
  </rm> 
</cluster>

```
同步配置:
要保证所有节点的ricci服务已启动，且ricci账户都配置密码
```shell
[root@node1-f30 ~]# cman_tool version -r
You have not authenticated to the ricci daemon on node1-f30.example.com
Password:
You have not authenticated to the ricci daemon on node3-f30.example.com
Password:
You have not authenticated to the ricci daemon on node2-f30.example.com
Password:
```
启动cman，rgmanager，modcluster服务
```shell
[root@node3-f30 ~]# /etc/init.d/cman start

Starting cluster:
Checking if cluster has been disabled at boot...    [ OK ]
Checking Network Manager...               [ OK ]
Global setup...                     [ OK ]
Loading kernel modules...                [ OK ]
Mounting configfs...                  [ OK ]
Starting cman...                    [ OK ]
Waiting for quorum...                  [ OK ]
Starting fenced...                   [ OK ]
Starting dlm_controld...                [ OK ]
Tuning DLM kernel config...               [ OK ]
Starting gfs_controld...                [ OK ]
Unfencing self...                    [ OK ]
Joining fence domain...                 [ OK ]

[root@node3-f30 ~]# /etc/init.d/rgmanager start
Starting Cluster Service Manager:             [ OK ]

[root@node3-f30 ~]# /etc/init.d/modclusterd start
Starting Cluster Module - cluster monitor: Setting verbosity level to LogBasic
[ OK ]
```
切换资源到node3节点上去运行，测试节点是否正常:
```shell
[root@node1-f30 ~]# clusvcadm -r httpd -m node3-f30.example.com
Trying to relocate service:httpd to node3-f30.example.com...Success
service:httpd is now running on node3-f30.example.com

同步fence_xvm.key:

[root@node1-f30 ~]# scp /etc/cluster/fence_xvm.key 172.25.30.12:/etc/cluster/fence_xvm.key
The authenticity of host '172.25.30.12 (172.25.30.12)' can't be established.
RSA key fingerprint is cf:7c:26:aa:4f:41:7b:21:5e:09:ce:8a:15:2c:97:32.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.25.30.12' (RSA) to the list of known hosts.
root@172.25.30.12's password:
fence_xvm.key                100% 512   0.5KB/s  00:00
```
Qdisk仲裁盘添加：

在新的机器上共享一个磁盘出来加入到集群中,共享磁盘需要用到scsi-target-utils工具包，可yum安装获得
```shell
[root@rhel6 ~]# yum install scsi*

将配置文件中的如下3行配置取消注释修改需要添加磁盘

[root@rhel6 ~]# vi /etc/tgt/targets.conf
<target iqn.2008-09.com.example:server.target1>
backing-store /dev/vdb1
</target>

集群的节点中安装包 iscsi-initiator-utils

[root@node1-f30 ~]# yum install scsi*

添加仲裁盘到本地,格式化仲裁盘

iscsiadm -m discovery -t st -p 172.25.30.13
iscsiadm -m node -l
mkqdisk -c /dev/sda -l myqdisk

[root@node3 ~]# iscsiadm -m discovery -t st -p 172.25.30.13
Starting iscsid:                      [ OK ]
172.25.30.13:3260,1 iqn.2008-09.com.example:server.target1

[root@node3 ~]# iscsiadm -m node -l
Logging in to [iface: default, target: iqn.2008-09.com.example:server.target1, portal: 172.25.30.13,3260] (multiple)
Login to [iface: default, target: iqn.2008-09.com.example:server.target1, portal: 172.25.30.13,3260] successful.

[root@node3 ~]# mkqdisk -c /dev/sda -l myqdisk
mkqdisk v3.0.12.1
Writing new quorum disk label 'myqdisk' to /dev/sda.
WARNING: About to destroy all data on /dev/sda; proceed [N/y] ? y
Warning: Initializing previously initialized partition
Initializing status block for node 1...
Initializing status block for node 2...
```
在luci图形界面中添加仲裁到集群

[![ha2](/images/ha2.png)](http://www.xiemx.com/wp-content/uploads/2016/01/ha2.png)
也可以通过修改配置文件然后通过cman_tool version -r来同步到集群中的每个节点上

配置文件:
```xml
<quorumd label="myqdisk" min_score="1">
<heuristic interval="5" program="ping 172.25.30.254 -c 1" tko="2"/>
</quorumd>
```
同步配置文件时要注意修改版本号