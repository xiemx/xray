---
layout: post
title: vcenter扩容Linux虚拟机磁盘
published: true
author: xiemx
comments: true
date: 2017-01-19 11:01:34
tags: [ vcenter ]
categories:
    - 'vcenter'
#permalink: '/2017/01/19/vcenter-expand-disk-to-linux'
---

![img](/images/img_58802ffca93b9.png)

![img](/images/img_5880322f10890.png)

磁盘扩容100G
1.vcenter扩容磁盘
vcenter增加步骤参考：[vcenter扩容window磁盘](/2017/01/12/vcenter-expand-disk-to-windows)

2.Linux需要rescan磁盘，重新读到大小
```shell
方法1:
reboot
方法2:
[root@localhost ~]# echo 1 > /sys/block/sdc/device/rescan
```
3.fdisk创建新分区
	注意创建新的磁盘后内核可能无法识别到分区表，需要运行partprobe刷新下分区表信息

4.lvm动态扩容
lvm参考：[lvm动态磁盘管理](http://www.xiemx.com/2015/11/29/linux动态磁盘管理（lvm）/)

完成后：

![img](/images/img_588031f4a62e1.png)

![img](/images/img_58803215d9370.png)

