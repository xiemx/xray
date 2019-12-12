---
layout: post
title: 三种共享存储比较
published: true
author: xiemx
comments: true
date: 2016-02-18 04:02:44
tags: [ ]
categories:
    - 'storage'
#permalink: '/2016/02/18/Three-shared-storage-comparisons'
---

### 共享存储（Share Storage）类型

  1. NAS(Network Attached Storage)网络附加存储
  2. SAN (Storage Area Network)储存区域网络
  3. iSAN (internet Storage Area Network )以太网存储区域网络,基于以太网的san

#### NAS(Network Attached Storage)
  1. 基于tcp/ip网络
  2. 以文件为单位进行操作（文件锁）

#### SAN (Storage Area Network)
  1. 基于硬盘驱动协议（sisc）传输的是磁道/扇区信息
  2. 基于扇区/block锁

#### iSAN (internet Storage Area Network )
  1. 基于tcp/ip网络 , 通过以太网数据包传递scsi协议数据
  2. 基于扇区/block锁