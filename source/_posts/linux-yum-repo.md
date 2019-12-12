---
layout: post
title: Linux包管理yum仓库配置
published: true
author: xiemx
comments: true
date: 2015-11-09 07:11:22
tags: [ linux, yum]
categories:
    - linux
---
Rhel中使用传统的rpm包安装会存在依存关系问题，虽然rpm为我们列出了依存关系，但是并没有为我们自动解决这个问题，依旧需要我们手动的去安装依存包。这样使得软件的安装非常麻烦，由此Linux中出现了yum工具，yum通过预读取软件的依存关系然后生存缓存，以后我们通过yum去安装软件是yum会根据缓存的关系表自动去识别rpm包的依存关系，为我们自动处理依存关系，使得软件的安装的工作变得简单。yum安装软件需要一个软件仓，yum会读取/etc/yum.repos.d/xxxxxxx.repo的仓库配置文件去仓库中提取软件来安装。我们可以通过修改仓配置文件，来修改仓库位置，使用本地或者速度快的仓，加速软件安装。

配置文件内容：
```
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
enabled=1
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6
```
以上是阿里云的开源镜像站的yum源配置。


配置文件中：
[ ]内的是仓库的名字
name是仓库的描述也可以说是名字
baseurl 仓库的位置
enabled 是否启用这个仓库，1为起用，0为禁用
gpgcheck 是否检查GPG签名（用来验证要安装的包是不是REDHAT官方的）
gpgkey gpgkey的存放地址我们开启gpgcheck=1才使用这项功能

由以上的配置文件我们可以看出只要修改baseurl我们就可以修改仓的位置，因此我们也可以使用本地的仓库。只要配置baseurl指向相对应的仓即可。如下
```
[xiemx] 
name=xiemx 
baseurl=file:///yumdatebase 
enabled=1 
gpgcheck=0
```
在运行下yum clear all清空下缓存 yum  makecache制作新的缓存。以上的baseurl我们是将系统安装盘的内容复制到本机的/yumdatebase下，由于是光盘的源系统由此也不需要校验程序的安全性所以关闭gpgcheck。也可以使用其它协议来链接仓库如ftp://xxxxxxxxx。


