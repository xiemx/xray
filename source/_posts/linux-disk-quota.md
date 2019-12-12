---
layout: post
title: linux磁盘配额
published: true
author: xiemx
comments: true
date: 2015-11-29 04:11:32
tags: [ linux, command ]
categories:
    - linux
---
再多用户的模式下，linux中常常需要对用户进行磁盘空间限制，例如虚拟主机需要限制用户的空间。linux大多数的发行版都采用quota来对磁盘配额来进行管理，quota是系统内核中的一个功能，要使用quota需要系统内核支持quota功能。目前使用的发行本中都是支持次功能的，如果内核不支持此功能那么就需要重新编译下内核来开启此功能
```shell
### grep CONFIG_QUOTA /boot/config-`uname -r` 来检查下内核是否支持

[root@localhost mnt]# grep CONFIG_QUOTA /boot/config-2.6.32-431.el6.x86_64
CONFIG_QUOTA=y
CONFIG_QUOTA_NETLINK_INTERFACE=y
# CONFIG_QUOTA_DEBUG is not set
CONFIG_QUOTA_TREE=y
CONFIG_QUOTACTL=y
```
如有以上两条则说明支持，另外quota是针对用户去限制其可使用的block和inode，所以应该是一个基于文件系统的配置，所以我们要在文件系统中开启quota功能，针对已经挂载过的文件系统我们可以使用
`mount -o  remount,usrquota,grpquota,default    /mnt` 
来重新挂载激活（mnt只是个举例，实际中先df 命令查看当前挂载信息，在选择要开启的文件系统），也可以写在`/etc/fstab`文件中`umount` 后再`mount -a` 或重启系统
```shell
[root@localhost mnt]# mount
/dev/mapper/VolGroup-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
/dev/mapper/vgxiemx-lv1 on /mnt type ext3 (rw)

[root@localhost mnt]# mount -o remount,grpquota,usrquota,defaults /mnt

[root@localhost mnt]# mount
/dev/mapper/VolGroup-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
/dev/mapper/vgxiemx-lv1 on /mnt type ext3 (rw,usrquota,grpquota)
[root@localhost mnt]#
```
**此时已开启/mnt对应的文件系统的quota功能，我们可针对/mnt目录下进行用户配额限制**


1. 检查文件系统的quota信息，生成quota数据库，会在文件系统的根目录下生成quota.user和quota.group二个data类型的文件可用file命令查看
```shell
[root@localhost mnt]# quotacheck -avug
quotacheck: Your kernel probably supports journaled quota but you are not using it. Consider switching to journaled quota to avoid running quotacheck after an unclean shutdown.
quotacheck: Scanning /dev/mapper/vgxiemx-lv1 [/mnt] done
quotacheck: Checked 3 directories and 6 files
[root@localhost mnt]# ls
aquota.group  aquota.user  lost+found
```
2. 启动quota
quotaon -av —a选项启动所有文件系统，也可指定具体某个文件系统。关闭`quotaoff`命令，参数相同。
```shell
[root@localhost /]# quotaon -av
/dev/mapper/vgxiemx-lv1 [/mnt]: group quotas turned on
/dev/mapper/vgxiemx-lv1 [/mnt]: user quotas turned on
```
3. 编辑用户的quota信息
edquota  -u testuser——会打开一个vi界面
```shell
Disk quotas for user testuser (uid 500):
Filesystem                                    blocks       soft       hard     inodes     soft     hard
/dev/mapper/vgxiemx-lv1                        0             10        100       0          10       100

filesystem：目标文件系统
block：现在已使用的block数量
inode：现在已使用的inode数量
soft：软限制，可使用的soft数量，0为不限制，具体数值设定后超过数值进行警告，但依旧可以写入，知道宽限时间到达后无法写入默认宽限7天
hard：硬限制，可使用block/inode数量，0为不限制，具体数值设定后超过数值，立即无法写入。
```
例：
```shell
[testuser@localhost mnt]$ dd if=/dev/zero of=/mnt/bigfile bs=1M count=20 ——生成一个20M的文件，查过soft block限制
dm-2: warning, user block quota exceeded.
1+0 records in
0+0 records out
98304 bytes (98 kB) copied, 0.00147599 s, 66.6 MB/s

[testuser@localhost mnt]$ du bigfile ——统计文件大小20M
20 bigfile

[testuser@localhost mnt]$ dd if=/dev/zero of=/mnt/bigfile bs=1M count=110——生成一个110M的文件，超过hard block限制，最后10M数据无法写入
dm-2: warning, user block quota exceeded.
dm-2: write failed, user block limit reached.
dd: writing `/mnt/bigfile': Disk quota exceeded
1+0 records in
0+0 records out
98304 bytes (98 kB) copied, 0.000795784 s, 124 MB/s

[testuser@localhost mnt]$ du bigfile——统计文件大小为100M
100 bigfile

[testuser@localhost mnt]$ ll
total 132
-rw-------. 1 root root 7168 Nov 22 04:01 aquota.group
-rw-------. 1 root root 7168 Nov 22 03:58 aquota.user
-rw-rw-r--. 1 testuser testuser 98304 Nov 22 04:02 bigfile
drwxrwxrwx. 2 root root 16384 Nov 22 02:31 lost+found

[testuser@localhost mnt]$ touch file{1..10}——创建10个文件，此时已有1个bigfile，总共11个文件超过soft inode限制
dm-2: warning, user file quota exceeded.

[testuser@localhost mnt]$ ls
aquota.group bigfile file10 file3 file5 file7 file9
aquota.user file1 file2 file4 file6 file8 lost+found

[testuser@localhost mnt]$ touch test{1..100}——创建100个文件，此时已有11个文件，所以最后的11个文件应该创建不成功
dm-2: write failed, user file limit reached.
touch: cannot touch `test90': Disk quota exceeded
touch: cannot touch `test91': Disk quota exceeded
touch: cannot touch `test92': Disk quota exceeded
touch: cannot touch `test93': Disk quota exceeded
touch: cannot touch `test94': Disk quota exceeded
touch: cannot touch `test95': Disk quota exceeded
touch: cannot touch `test96': Disk quota exceeded
touch: cannot touch `test97': Disk quota exceeded
touch: cannot touch `test98': Disk quota exceeded
touch: cannot touch `test99': Disk quota exceeded
touch: cannot touch `test100': Disk quota exceeded

[testuser@localhost mnt]$ ls
aquota.group file7 test15 test24 test33 test42 test51 test60 test7 test79 test88
aquota.user file8 test16 test25 test34 test43 test52 test61 test70 test8 test89
bigfile file9 test17 test26 test35 test44 test53 test62 test71 test80 test9
file1 lost+found test18 test27 test36 test45 test54 test63 test72 test81
file10 test1 test19 test28 test37 test46 test55 test64 test73 test82
file2 test10 test2 test29 test38 test47 test56 test65 test74 test83
file3 test11 test20 test3 test39 test48 test57 test66 test75 test84
file4 test12 test21 test30 test4 test49 test58 test67 test76 test85
file5 test13 test22 test31 test40 test5 test59 test68 test77 test86
file6 test14 test23 test32 test41 test50 test6 test69 test78 test87
```

4.设置宽限时间
```
edquota -t  ——默认7天
Grace period before enforcing soft limits for users:
Time units may be: days, hours, minutes, or seconds
Filesystem                                   Block grace period                        Inode grace period
/dev/mapper/vgxiemx-lv1                      7days                                     7days
```
5.查询所有用户的磁盘配额情况，用户也可用quota查询当前用户的quota信息
```
repquota -a

[root@localhost mnt]# repquota -a
*** Report for user quotas on device /dev/mapper/vgxiemx-lv1
Block grace time: 7days; Inode grace time: 7days
Block limits File limits
User                      used      soft      hard    grace    used      soft      hard       grace
-------------------------------------------------------------------------------------------
root           --        956592     0         0                 6          0        0
testuser       ++        100        10        100      6days    100        10       100       6days
```
*如果要设置quota，建议在/etc/rc.d/rc.local中写入quotacheck和quotaon 已保证系统启动时quota服务开启且数据都是最新的。分区的quote功能开启建议在/etc/fstab中。*

例：
```
/etc/fstab
# Created by anaconda on Sun May 24 08:16:25 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup-lv_root / ext4 defaults 1 1
UUID=3fe6311e-b3fe-4f7f-8558-4f011eab1dde /boot ext4 defaults 1 2
/dev/mapper/VolGroup-lv_swap swap swap defaults 0 0
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5,mode=620 0 0
sysfs /sys sysfs defaults 0 0
proc /proc proc defaults 0 0
/dev/mapper/vgxiemx-lv1 /mnt ext3 defaults,usrquota,grpquota 0 0
```