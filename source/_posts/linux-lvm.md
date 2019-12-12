---
layout: post
title: Linux动态磁盘管理（lvm）
published: true
author: xiemx
comments: true
date: 2015-11-29 02:11:11
tags: [ linux, lvm]
categories:
    - linux
---
lvm是将底层硬件存储屏蔽整合，通过软件组合在对系统开放，此时系统查看到的就不是底层一块块的物理磁盘了而是我们虚拟的逻辑磁盘，我们可以自由的对这些磁盘进行容量管理。lvm从系统层面到物理层面分别对应“文件系统——lv——vg——pv——磁盘”。所有对磁盘的管理都要遵循这个物理结构的顺序来处理。创建时由下而上，删除时由上而下，调整中间部分是也要结合上下部分分析需求具体在进行处理。

pv——物理卷
vg——卷组
lv——逻辑卷
pe——最小存储单元，类似与磁盘block

lvm是介于系统与硬件之间的一种磁盘组织方法，下层为上层的基础，下层制约上层。

## lvm管理

1. 磁盘分区
```
fdisk ／dev/vda
```
2. 同步内核中的分区表信息（如果是新硬盘一般不需要这一部分）
```
partx  -a   /dev/vda
```
3. 创建pv
```
pvcreate ／dev/vda1  ——有多个分区时可以依次空格间隔写在后面，也可以针对整块硬盘去直接创建

pvs和pvdisplay查看pv缩略和详细信息

[root@localhost ~]# pvs
PV VG Fmt Attr PSize PFree
/dev/sda2 VolGroup lvm2 a-- 19.51g 0
/dev/sdb lvm2 a-- 10.00g 10.00g

[root@localhost ~]# pvdisplay
--- Physical volume ---
PV Name               /dev/sda2
VG Name               VolGroup
PV Size               19.51 GiB / not usable 3.00 MiB
Allocatable           yes (but full)
PE Size               4.00 MiB
Total PE              4994
Free PE               0
Allocated PE          4994
PV UUID               fcQLJh-oq2G-adfr-QeEK-sdtl-3XND-m1YtPA

"/dev/sdb" is a new physical volume of "10.00 GiB"
--- NEW Physical volume ---
PV Name               /dev/sdb
VG Name
PV Size               10.00 GiB
Allocatable           NO
PE Size               0
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               WQpkNi-1cN6-QogB-TLB2-R2wT-Agmh-BRBiUN
```
4. 创建vg
```
vgcreate  -s 8M vgname /dev/vda1  /dev/vda2  —— s选项制定pe的大小默认为4M

vgs和vgdisplay查看vg信息

[root@localhost ~]# vgcreate -s 8M vgxiemx /dev/sdb
Volume group "vgxiemx" successfully created
[root@localhost ~]# vgs
VG       #PV #LV #SN Attr   VSize  VFree
VolGroup   1   2   0 wz--n- 19.51g    0
vgxiemx    1   0   0 wz--n-  9.99g 9.99g
[root@localhost ~]# vgdisplay
--- Volume group ---
VG Name               vgxiemx
System ID
Format                lvm2
Metadata Areas        1
Metadata Sequence No  1
VG Access             read/write
VG Status             resizable
MAX LV                0
Cur LV                0
Open LV               0
Max PV                0
Cur PV                1
Act PV                1
VG Size               9.99 GiB
PE Size               8.00 MiB
Total PE              1279
Alloc PE / Size       0 / 0
Free  PE / Size       1279 / 9.99 GiB
VG UUID               5WZfct-snfo-aEDl-gHuO-QDbz-v83j-L0lwmR
```
5. 创建lv
```
lvcreate -l 100 -n lvname vgname——l选项指定分配100个pe给lv ，n指定名称，最后从哪个vg创建。

lvcreate -L1000M -n lvname vgname——L便是直接分配多大空间单位为K M G。L和l参数还有很多种容量表示方法man查阅

lvs和lvdisplay查看lv信息

[root@localhost ~]# lvcreate -n lv1 -l 1279 vgxiemx
Logical volume "lv1" created
[root@localhost ~]# lvs
LV VG Attr LSize Pool Origin Data% Move Log Cpy%Sync Convert
lv_root VolGroup -wi-ao---- 17.51g
lv_swap VolGroup -wi-ao---- 2.00g
lv1 vgxiemx -wi-a----- 9.99g
[root@localhost ~]# lvdisplay
--- Logical volume ---
LV Path /dev/vgxiemx/lv1
LV Name lv1
VG Name vgxiemx
LV UUID nViafR-nrnS-REMQ-bGOT-uhDe-Skd0-XUNwmU
LV Write Access read/write
LV Creation host, time localhost.localdomain, 2015-11-22 01:50:41 +0800
LV Status available
# open 0
LV Size 9.99 GiB
Current LE 1279
Segments 1
Allocation inherit
Read ahead sectors auto
- currently set to 256
Block device 253:2
```
6. 创建文件系统
```
mkfs.ext3  /dev/vgname/lvname ——创建文件系统

[root@localhost ~]# mkfs -t ext3 /dev/vgxiemx/lv1
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
655360 inodes, 2619392 blocks
130969 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2684354560
80 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 30 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```
7. 挂载lv
```
mount /dev/vgname/lvname  /mnt

[root@localhost ~]# mount /dev/vgxiemx/lv1 /mnt
[root@localhost mnt]# df -l
Filesystem 1K-blocks Used Available Use% Mounted on
/dev/mapper/VolGroup-lv_root 18069936 6767876 10384148 40% /
tmpfs 953788 72 953716 1% /dev/shm
/dev/sda1 495844 39915 430329 9% /boot
/dev/mapper/vgxiemx-lv1 10313016 154232 9634908 2% /mnt
```
## lvm调整

### pv的新增、删除

#### pv的新增
就是增加新的空闲分区或者新的硬盘创建成新的空闲pv同上pv创建命令一致

#### pv删除
```
pvremove /dev/vdb

[root@localhost /]# pvremove /dev/sdb
Labels on physical volume "/dev/sdb" successfully wiped
[root@localhost /]# pvs
PV VG Fmt Attr PSize PFree
/dev/sda2 VolGroup lvm2 a-- 19.51g 0
```
#### 移动pv中的数据

由下图我们可以看到pv/dev/sdb1中的1G数据已经被我们移动到/dev/sdb2中去了，如果是单硬盘做的pv此时我们就可以撤除/dev/sdb1对应的这块硬盘，而数据没有丢失
```
pvremove /dev/sdb1 /dev/sdb2

[root@localhost mnt]# pvs
PV         VG       Fmt  Attr PSize  PFree
/dev/sda2  VolGroup lvm2 a--  19.51g    0
/dev/sdb1  vgxiemx  lvm2 a--   1.01g 8.00m
/dev/sdb2  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb3  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb5           lvm2 a--   2.01g 2.01g
/dev/sdb6           lvm2 a--   4.96g 4.96g
[root@localhost mnt]# pvmove /dev/sdb1 /dev/sdb2
/dev/sdb1: Moved: 1.6%
/dev/sdb1: Moved: 45.3%
/dev/sdb1: Moved: 89.8%
/dev/sdb1: Moved: 100.0%
[root@localhost mnt]# pvs
PV         VG       Fmt  Attr PSize  PFree
/dev/sda2  VolGroup lvm2 a--  19.51g    0
/dev/sdb1  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb2  vgxiemx  lvm2 a--   1.01g 8.00m
/dev/sdb3  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb5           lvm2 a--   2.01g 2.01g
/dev/sdb6           lvm2 a--   4.96g 4.96g
```
### vg扩容、缩小、删除

#### vg的扩容

vg的容量来自于pv，如果vg的容量不够那么vg扩容其实就是新增空闲的pv到vg中去
```
vgextend  /dev/sdb5  vgxiemx

[root@localhost /]# pvs
PV VG Fmt Attr PSize PFree
/dev/sda2 VolGroup lvm2 a-- 19.51g 0
/dev/sdb1 vgxiemx lvm2 a-- 1.01g 1.01g
/dev/sdb2 vgxiemx lvm2 a-- 1.01g 1.01g
/dev/sdb3 vgxiemx lvm2 a-- 1.01g 1.01g
/dev/sdb5                 lvm2 a-- 2.01g 2.01g
/dev/sdb6                 lvm2 a-- 4.96g 4.96g
[root@localhost /]# vgextend vgxiemx /dev/sdb5
Volume group "vgxiemx" successfully extended
[root@localhost /]# pvs
PV VG Fmt Attr PSize PFree
/dev/sda2 VolGroup lvm2 a-- 19.51g 0
/dev/sdb1 vgxiemx lvm2 a-- 1.01g 1.01g
/dev/sdb2 vgxiemx lvm2 a-- 1.01g 1.01g
/dev/sdb3 vgxiemx lvm2 a-- 1.01g 1.01g
/dev/sdb5 vgxiemx lvm2 a-- 2.00g 2.00g
/dev/sdb6                 lvm2 a-- 4.96g 4.96g
```
#### vg的减小

也就是将vg中的pv移除，但pv移除是存在pv中的pe都已经被使用存在数据，这是我们就需要移动要删除的数据到新的pv中去，新的pv空间必须要比旧的pv空间大。
```
vgreduce vgxiemx /dev/sdb5

[root@localhost /]# pvs
PV         VG       Fmt  Attr PSize  PFree
/dev/sda2  VolGroup lvm2 a--  19.51g    0
/dev/sdb1  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb2  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb3  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb5  vgxiemx  lvm2 a--   2.00g 2.00g
/dev/sdb6           lvm2 a--   4.96g 4.96g
[root@localhost /]# vgreduce vgxiemx /dev/sdb5
Removed "/dev/sdb5" from volume group "vgxiemx"
[root@localhost /]# pvs
PV         VG       Fmt  Attr PSize  PFree
/dev/sda2  VolGroup lvm2 a--  19.51g    0
/dev/sdb1  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb2  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb3  vgxiemx  lvm2 a--   1.01g 1.01g
/dev/sdb5           lvm2 a--   2.01g 2.01g
/dev/sdb6           lvm2 a--   4.96g 4.96g
```
#### vg的删除

```
vgremove vgxiemx

[root@localhost /]# vgremove vgxiemx
Volume group "vgxiemx" successfully removed
[root@localhost /]# vgs
VG #PV #LV #SN Attr VSize VFree
VolGroup 1 2 0 wz--n- 19.51g 0
```
### lv的扩容、缩小、删除

#### lv的缩小
```
lvreduce -L 9000M /dev/vgxiemx/lv1

[root@localhost /]# lvreduce -L 9000M /dev/vgxiemx/lv1
WARNING: Reducing active logical volume to 8.79 GiB
THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce lv1? [y/n]: y
Reducing logical volume lv1 to 8.79 GiB
Logical volume lv1 successfully resized
[root@localhost /]# lvs
LV VG Attr LSize Pool Origin Data% Move Log Cpy%Sync Convert
lv_root VolGroup -wi-ao---- 17.51g
lv_swap VolGroup -wi-ao---- 2.00g
lv1 vgxiemx -wi-a----- 8.79g
```
#### lv的扩容
```
lvextend -L 10000M /dev/vgxiemx/lv1

[root@localhost /]# lvextend -L 10000M /dev/vgxiemx/lv1
Extending logical volume lv1 to 9.77 GiB
Logical volume lv1 successfully resized
[root@localhost /]# lvs
LV VG Attr LSize Pool Origin Data% Move Log Cpy%Sync Convert
lv_root VolGroup -wi-ao---- 17.51g
lv_swap VolGroup -wi-ao---- 2.00g
lv1 vgxiemx -wi-a----- 9.77g
```
#### lv的删除
```
lvremove /dev/vgxiemx/lv1

[root@localhost /]# lvremove /dev/vgxiemx/lv1
Do you really want to remove active logical volume lv1? [y/n]: y
Logical volume "lv1" successfully removed
[root@localhost /]# lvs
LV VG Attr LSize Pool Origin Data% Move Log Cpy%Sync Convert
lv_root VolGroup -wi-ao---- 17.51g
lv_swap VolGroup -wi-ao---- 2.00g
```
### 文件系统的扩缩

#### 检测文件系统
```
e2fsck /dev/vgxiemx/lv1

[root@localhost /]# e2fsck -f /dev/vgxiemx/lv1
e2fsck 1.41.12 (17-May-2010)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vgxiemx/lv1: 11/655360 files (0.0% non-contiguous), 79696/2619392 blocks
```
#### 调整文件系统大小
```
resize2fs /dev/vgxiemx/lv1 9000M

[root@localhost /]# resize2fs /dev/vgxiemx/lv1 9000M
resize2fs 1.41.12 (17-May-2010)
Resizing the filesystem on /dev/vgxiemx/lv1 to 2304000 (4k) blocks.
The filesystem on /dev/vgxiemx/lv1 is now 2304000 blocks long.
```
 