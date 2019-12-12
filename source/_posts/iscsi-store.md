---
layout: post
title: ISCSI存储网络构建
published: true
author: xiemx
comments: true
date: 2016-02-18 04:02:11
tags: [ iscsi, linux, store ]
categories:
    - linux
---
iscsi的结构和san的结构

[![san](/images/san.png)](http://www.xiemx.com/wp-content/uploads/2016/02/san.png)

#### iSCSI 通信端
* 发起 I/O 请求的启动设备(Initiator)
* 响应请求并执行实际 I/O 操作的目标设备(Target)

#### iSCSI工作过程

* target端导出共享设备
* initiator端发现设备
* initiator端导入设备
* initiator端分区、格式化、挂接设备

#### iSCSI实现套件
服务端（target）：scsi-target-utils软件包
客户端（initiator）：iscsi-initiator-utils软件包

### 部署过程：
```
yum insitall scsi-target-utils
yum insitall iscsi-initiator-utils
```
服务器端导出共享设备： 配置文件如下
```
[root@rhel6 ~]# grep -v '#\|^$' /etc/tgt/targets.conf
default-driver iscsi
<target iqn.2015-10.com.example-f30:vdb1-1g>
  backing-store /dev/vdb1
</target>
```
 启动tgtd服务
```shell
[root@rhel6 ~]# /etc/init.d/tgtd start
Starting SCSI target daemon:                [ OK ]

 ```

查看客户端本地有无/dev/sda设备
```shell
[root@rhel6 ~]# ll /dev/sda
ls: cannot access /dev/sda: No such file or directory
```
搜索target端的共享设备
```shell
[root@rhel6 ~]# iscsiadm -m discovery -t st -p 172.25.30.10
172.25.30.10:3260,1 iqn.2015-10.com.example-f30:vdb1-1g
```
 导入target端的共享设备
```shell
[root@rhel6 ~]# iscsiadm -m node -l
Logging in to [iface: default, target: iqn.2015-10.com.example-f30:vdb1-1g, portal: 172.25.30.10,3260] (multiple)
Login to [iface: default, target: iqn.2015-10.com.example-f30:vdb1-1g, portal: 172.25.30.10,3260] successful.
```
 查看本地/dev/sda设备
```shell
[root@rhel6 ~]# ll /dev/sda
brw-rw----. 1 root disk 8, 0 Jan 14 14:51 /dev/sda
```
卸载导入的设备
```shell
[root@rhel6 nodes]# ll /dev/sd*
brw-rw----. 1 root disk 8, 0 Jan 14 14:53 /dev/sda
brw-rw----. 1 root disk 8, 16 Jan 14 15:35 /dev/sdb
brw-rw----. 1 root disk 8, 17 Jan 14 15:35 /dev/sdb1

[root@rhel6 nodes]# iscsiadm -m node -T iqn.2016-01-14.com.example.node4-f24:vdb1-1G -u
Logging out of session [sid: 2, target: iqn.2016-01-14.com.example.node4-f24:vdb1-1G, portal: 172.25.24.13,3260]
Logout of [sid: 2, target: iqn.2016-01-14.com.example.node4-f24:vdb1-1G, portal: 172.25.24.13,3260] successful.

[root@rhel6 nodes]# ll /dev/sd*
brw-rw----. 1 root disk 8, 0 Jan 14 14:53 /dev/sda
```
### 配置iSCSI的acl和验证

在target主配置文件中添加如下两行
```
[root@rhel6 ~]# grep -v '#\|^$' /etc/tgt/targets.conf
default-driver iscsi
<target iqn.2015-10.com.example-f30:vdb1-1g>
  backing-store /dev/vdb1
</target>

<target iqn.2015-10.com.example-f30:vdb2-2g>
  backing-store /dev/vdb2
  initiator-address 172.25.30.0/24
  incominguser xiemx uplooking
</target>
```
重新加载配置文件
```
/etc/init.d/tgtd   reload
/etc/init.d/tgtd   force-reload
```
 

查看配置导出的设备信息
```shell
[root@rhel6 ~]# tgtadm --lld iscsi --mode target --op show
Target 1: iqn.2015-10.com.example-f30:vdb1-1g
  System information:
​    Driver: iscsi
​    State: ready
  I_T nexus information:
  LUN information:
​    LUN: 0
​      Type: controller
​      SCSI ID: IET   00010000
​      SCSI SN: beaf10
​      Size: 0 MB, Block size: 1
​      Online: Yes
​      Removable media: No
​      Prevent removal: No
​      Readonly: No
​      Backing store type: null
​      Backing store path: None
​      Backing store flags:
​    LUN: 1
​      Type: disk
​      SCSI ID: IET   00010001
​      SCSI SN: beaf11
​      Size: 1074 MB, Block size: 512
​      Online: Yes
​      Removable media: No
​      Prevent removal: No
​      Readonly: No
​      Backing store type: rdwr
​      Backing store path: /dev/vdb1
​      Backing store flags:
  Account information:
  ACL information:
​    ALL
 Target 2: iqn.2015-10.com.example-f30:vdb2-2g
  System information:
​     Driver: iscsi
​     State: ready
  I_T nexus information:
  LUN information:
​    LUN: 0
​      Type: controller
​      SCSI ID: IET   00020000
​      SCSI SN: beaf20
​      Size: 0 MB, Block size: 1
​      Online: Yes
​      Removable media: No
​      Prevent removal: No
​      Readonly: No
​      Backing store type: null
​      Backing store path: None
​      Backing store flags:
​    LUN: 1
​      Type: disk
​      SCSI ID: IET   00020001
​      SCSI SN: beaf21
​      Size: 2148 MB, Block size: 512
​      Online: Yes
​      Removable media: No
​      Prevent removal: No
​      Readonly: No
​      Backing store type: rdwr
​      Backing store path: /dev/vdb2
​      Backing store flags:
  Account information:
​     xiemx
  ACL information:
​    172.25.30.0/24

```

initiator端开启CHAP验证并配置用户密码

```shell
[root@rhel6 ~]grep -v '^#\|^$' /etc/iscsi/iscsid.conf
iscsid.startup = /etc/rc.d/init.d/iscsid force-start
node.startup = automatic
node.leading_login = No
node.session.auth.authmethod = CHAP
node.session.auth.username = xiemx
node.session.auth.password = uplooking
node.session.timeo.replacement_timeout = 120
node.conn[0].timeo.login_timeout = 15
node.conn[0].timeo.logout_timeout = 15
node.conn[0].timeo.noop_out_interval = 5
node.conn[0].timeo.noop_out_timeout = 5
node.session.err_timeo.abort_timeout = 15
node.session.err_timeo.lu_reset_timeout = 30
node.session.err_timeo.tgt_reset_timeout = 30
node.session.initial_login_retry_max = 8
node.session.cmds_max = 128
node.session.queue_depth = 32
node.session.xmit_thread_priority = -20
node.session.iscsi.InitialR2T = No
node.session.iscsi.ImmediateData = Yes
node.session.iscsi.FirstBurstLength = 262144
node.session.iscsi.MaxBurstLength = 16776192
node.conn[0].iscsi.MaxRecvDataSegmentLength = 262144
node.conn[0].iscsi.MaxXmitDataSegmentLength = 0
discovery.sendtargets.iscsi.MaxRecvDataSegmentLength = 32768
node.conn[0].iscsi.HeaderDigest = None
node.session.nr_sessions = 1
node.session.iscsi.FastAbort = Yes

```

initiator重新发现并导入共享设备

```shell
[root@rhel6 nodes]# iscsiadm -m discovery -t st -p 172.25.30.10
172.25.30.10:3260,1 iqn.2015-10.com.example-f30:vdb1-1g
172.25.30.10:3260,1 iqn.2015-10.com.example-f30:vdb2-2g

[root@rhel6 nodes]# iscsiadm -m node -T iqn.2015-10.com.example-f30:vdb1-1g -l
Logging in to [iface: default, target: iqn.2015-10.com.example-f30:vdb1-1g, portal: 172.25.30.10,3260] (multiple)
Login to [iface: default, target: iqn.2015-10.com.example-f30:vdb1-1g, portal: 172.25.30.10,3260] successful.

[root@rhel6 nodes]# iscsiadm -m node -T iqn.2015-10.com.example-f30:vdb2-2g -l
Logging in to [iface: default, target: iqn.2015-10.com.example-f30:vdb2-2g, portal: 172.25.30.10,3260] (multiple)
Login to [iface: default, target: iqn.2015-10.com.example-f30:vdb2-2g, portal: 172.25.30.10,3260] successful.

[root@rhel6 nodes]# ll /dev/sd*
brw-rw----. 1 root disk 8, 16 Jan 14 16:04 /dev/sdb
brw-rw----. 1 root disk 8, 32 Jan 14 16:04 /dev/sdc
```