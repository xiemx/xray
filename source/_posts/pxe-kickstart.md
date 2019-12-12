---
layout: post
title: PXE批量在线安装操作系统
published: true
author: xiemx
comments: true
date: 2016-01-17 07:01:10
tags: [ pxe, kickstart, linux ]
categories:
    - linux
# permalink: '/2016/01/17/pxe%e6%89%b9%e9%87%8f%e5%9c%a8%e7%ba%bf%e5%ae%89%e8%a3%85%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f'
---
pxe 通过网络方式安装部署

* dhcp:动态管理协议
* 网卡支持tftp(文件下载使用,不支持验证，安全系数低,只下载些基础文件）使用http下载ks.conf(应答文件)/rpm包

### 流程
```
客户端向服务端申请下载 dhcp.client  服务端返回信息客户端下载，并分配给客户端ip地址
客户端拿到ip地址后去tftp服务端下载pxelinux.0 并在客户端安装pxelinux.0(引导安装程序）
客户端去下载配置文件 pxelinux.cfg/default 安装后出现安装标签 指引安装（例 install foution0)
选择安装标签后 到服务端下载unlinuz(微型运行平台） initrd(基本的命令 程序） ks文件路径（自动安装使用）
然后出现安装界面 开始交互式安装
如果自动安装去找http服务器，下载应答ks.cfg,下载rpm包 自动安装后会再此运行ks.cfg脚本
```
 

### 原理图：
```
clients：dhcp client
------------------------------> dhcp server
<------------------------------
分配地址池ip，告知tftp的地址
请求pxelinux.0文件
------------------------------> tftp server
<------------------------------
pxelinux.0
引导界面
-------------------------------> tftp server
<------------------------------
pxelinux.cfg/default
安装界面
-------------------------------> tftp server
<-------------------------------
vmlinuxz initrd ks文件路径

-------------------------------> http server
<------------------------------
ks.cfg rpm包
<-------------------------------
自动执行脚本
安装完成
```
### 操作步骤

依据以上原理图可以得知PXE过程需要用到的文件有pxelinux.o、default、ks.cfg、vmlinuxz、initrd.img。需要用到的协议tftp、dhcp、http。

pxelinux.0 来源syslinux软件包
ks.cfg 来源kickstart，也可以通过安装system-config-kickstart来图形化配置
vmlinuz 来源于iso镜像文件
default 需要手工配置
initrd.img 来源iso镜像文件

1.安装软件
```
yum install httpd dhcp tftp-server -y
```
2.配置dhcp
```
[root@linux]# cat /etc/dhcpd.conf
ddns-update-style interim;
allow booting; #定义能够PXE启动
allow bootp; #定义支持bootp
next-server 192.168.0.1; #TFTP Server的IP地址
filename "pxelinux.0"; #bootstrap 文件(NBP)

default-lease-time 1800;
max-lease-time 7200;
ping-check true;
option domain-name-servers 192.168.0.1;

subnet 192.168.0.0 netmask 255.255.255.0
{
range 192.168.0.128 192.168.0.220;
option routers 192.168.0.1;
option broadcast-address 192.168.0.255;
}
```
3.启动tftp
```
[root@linux]# cat /etc/xinetd.d/tftp
service tftp
{
socket_type = dgram
protocol = udp
wait = yes
user = root
server = /usr/sbin/in.tftpd
server_args = -s /var/lib/tftpboot   tftp服务根目录
disable = no   是否关闭tftp服务
per_source = 11
cps = 100 2
flags = IPv4
}

重启xinetd服务
```
4.获取pxelinux.0文件
```
[root@linux]# rpm -ql syslinux | grep "pxelinux.0"
/usr/lib/syslinux/pxelinux.0
[root@linux]# cp /usr/lib/syslinux/pxelinux.0 /var/lib/tftpboot/
```
5.创建/var/lib/tftp/pxelinux.cfg目录、创建default文件
```
将 boot.msg initrd.img splash.png vesamenu.c32 vmlinuz 复制到/var/lib/tftpboot/

[root@linux]# cat /tftpboot/pxelinux.cfg/default

default vesamenu.c32   显示图形化引导界面，也可以写成default linux文本化界面
timeout 60     等待操作时间
display boot.msg    显示一些引导信息

menu background splash.jpg   背景图片
menu title Welcome to pxe Setup!   界面标题

label 1
menu label Boot from ^local drive   安装1选项标题
menu default    60s无操作默认启动此选项
localboot 0xffff

label 2
menu label Install linux
ipappend 2
kernel vmlinuz
append initrd=initrd.img ks=http://172.25.16.9/ks.cfg
```
6.将iso文件展开到http目录下

7.生成ks.cfg文件

  `system-config-kickstart`安装此图形化工具，生成自动应答脚本。也可以拷贝已安装系统中自动生成的脚本。

8.关闭防火墙，selinux。测试http、tftp等服务都正常启动且文件都已放置可正常访问下载。