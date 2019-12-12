---
layout: post
title: Linux启动过程
published: true
author: xiemx
comments: true
date: 2015-11-15 04:11:22
tags: [ linux ]
categories:
    - linux
---
1. 加载自检（post）、BIOS
  
当你打开计算机电源，计算机会对硬件设备加电，然后去检查cpu，内存，主板这些硬件设备。加点通过后加载BIOS信息，读取bios中存储的设备启动顺序信息，寻找启动设备一般都是硬盘、光盘、U盘等。

2. 读取MBR
  
硬盘上第0磁道第一个扇区被称为MBR，也就是Master Boot Record，即主引导记录，大小是512字节，存放了446字节的boots loader、64字节的分区表信息和2字节的硬盘有效标记。
  
系统找到BIOS所指定的硬盘的MBR后，就会将其复制到0×7c00地址所在的物理内存中。其实被复制到物理内存的内容就是Boot Loader，常用的boot loader 有grub、lilo。

3. Boot Loader
  
系统读取内存中的grub配置信息（一般为menu.lst或grub.lst），并依照此配置信息来启动不同的操作系统。

4. 加载内核
  
依据grub.conf配置文件中的配置文件，系统读取内核，挂载虚拟文件系统initrd。

5. 启动init
  
内核被加载后运行第一个进程/sbin/init，读取配置文件/etc/init/rc.Scanf,根据此文件配置在找到/etc/rc.d/rc.sysinit开启系统初始化，在读取/etc/inittab文件，确定系统的启动级别。

6. 运行inittab中规定级别的脚本程序
  
根据运行级别的不同，系统会运行rc0.d到rc6.d中的相应的脚本程序，/etc/rc.d/rcX.d/目录。

7. 执行/etc/rc.d/rc.local
  
/etc/rc.d/rcX.d/中的脚本按照配置启动完成后，会启动rc.local。
  
8. 执行/bin/login
  
调用登录程序，启动登陆界面。

以上只是简单的系统启动过程，具体详细的系统层面的启动从第boot loader之后的执行都可以参看系统中的配置文件。可以读取相对应的脚本和配置代码，根据文件中的规定一步步的理解系统的启动，理解系统启动的详细步骤。
