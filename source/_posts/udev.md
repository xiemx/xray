---
layout: post
title: UDEV规则
published: true
author: xiemx
comments: true
date: 2016-02-18 05:02:23
tags: [ udev, linux ]
categories:
    - linux
---
udev根据系统中的硬件设备的状态动态更新设备，通过对内核产生的设备名增加别名的方式来达到不管设备连接的顺序而维持一个统一的设备名的目的。udev可以根据设备的其他信息如总线（bus），生产商（vendor）等不同来区分不同的设备，并产生设备文件。udev是硬件平台无关的，属于user space的进程，它脱离驱动层的关联而建立在操作系统之上，遵循Linux Standard Base （LSB）设备命名方法，但也可以自定义命名

### 工作流程图

![image-20191018151450374](/images/image-20191018151450374.png)

### 配置文件、rule参数

```conf
[mingxu.xie@cn-aux-cc udev]$ ll -R
.:
total 8
drwxr-xr-x 2 root root 4096 Jun  6 02:58 rules.d
-rw-r--r-- 1 root root  218 Mar 19  2014 udev.conf

./rules.d:
total 32
-rw-r--r-- 1 root root  640 Jul 31  2016 51-ec2-hvm-devices.rules
-rw-r--r-- 1 root root  641 Jul 31  2016 52-ec2-vcpu.rules
-rw-r--r-- 1 root root  740 Jul 31  2016 53-ec2-network-interfaces.rules
-rw-r--r-- 1 root root  680 Jul 31  2016 60-cdrom_id.rules
-rw-r--r-- 1 root root  326 Apr 26  2016 60-raw.rules
-rw-r--r-- 1 root root 1343 Jul 31  2016 70-ec2-nvme-devices.rules
-rw-r--r-- 1 root root 1424 Jul 31  2016 75-persistent-net-generator.rules
-rwxr-xr-x 1 root root  343 Aug 21  2016 80-docker.rules

# 主配置 /etc/udev/udev.conf 
udev_root="/dev/"
udev_rules="/etc/udev/rules.d/"

#rule规则的文件命名第一段为执行顺序，同rc脚本
[mingxu.xie@cn-aux-cc udev]$ cat rules.d/53-ec2-network-interfaces.rules
# Copyright (C) 2012 Amazon.com, Inc. or its affiliates.
# All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License").
# You may not use this file except in compliance with the License.
# A copy of the License is located at
#
#    http://aws.amazon.com/apache2.0/
#
# or in the "license" file accompanying this file. This file is
# distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS
# OF ANY KIND, either express or implied. See the License for the
# specific language governing permissions and limitations under the
# License.

ACTION=="add", SUBSYSTEM=="net", KERNEL=="eth*", IMPORT{program}="/bin/sleep 1"
SUBSYSTEM=="net", KERNEL=="eth*", RUN+="/etc/sysconfig/network-scripts/ec2net.hotplug"
```
```conf
#监听一个docker run -it --rm nginx bash容器启动的设备变化情况，如下
[mingxu.xie@cn-aux-cc udev]$ udevadm monitor
monitor will print the received events for:
UDEV - the event which udev sends out after rule processing
KERNEL - the kernel uevent

KERNEL[240932.229041] add      /devices/virtual/bdi/253:8 (bdi)
KERNEL[240932.229579] add      /devices/virtual/block/dm-8 (block)
UDEV  [240932.229612] add      /devices/virtual/bdi/253:8 (bdi)
UDEV  [240932.229641] add      /devices/virtual/block/dm-8 (block)
KERNEL[240932.248388] change   /devices/virtual/block/dm-8 (block)
UDEV  [240932.272904] change   /devices/virtual/block/dm-8 (block)
KERNEL[240932.425971] remove   /devices/virtual/block/dm-8 (block)
UDEV  [240932.428073] remove   /devices/virtual/block/dm-8 (block)
KERNEL[240932.456141] remove   /devices/virtual/bdi/253:8 (bdi)
KERNEL[240932.456261] remove   /devices/virtual/block/dm-8 (block)
UDEV  [240932.456380] remove   /devices/virtual/bdi/253:8 (bdi)
UDEV  [240932.456419] remove   /devices/virtual/block/dm-8 (block)
KERNEL[240932.469307] add      /devices/virtual/bdi/253:8 (bdi)
KERNEL[240932.469384] add      /devices/virtual/block/dm-8 (block)
UDEV  [240932.469657] add      /devices/virtual/bdi/253:8 (bdi)
UDEV  [240932.469691] add      /devices/virtual/block/dm-8 (block)
KERNEL[240932.488228] change   /devices/virtual/block/dm-8 (block)
UDEV  [240932.504203] change   /devices/virtual/block/dm-8 (block)
KERNEL[240932.636785] remove   /devices/virtual/block/dm-8 (block)
UDEV  [240932.638755] remove   /devices/virtual/block/dm-8 (block)
KERNEL[240932.676137] remove   /devices/virtual/bdi/253:8 (bdi)
KERNEL[240932.676245] remove   /devices/virtual/block/dm-8 (block)
UDEV  [240932.676367] remove   /devices/virtual/bdi/253:8 (bdi)
UDEV  [240932.676420] remove   /devices/virtual/block/dm-8 (block)
KERNEL[240932.683098] add      /devices/virtual/bdi/253:8 (bdi)
KERNEL[240932.683204] add      /devices/virtual/block/dm-8 (block)
UDEV  [240932.683384] add      /devices/virtual/bdi/253:8 (bdi)
UDEV  [240932.683582] add      /devices/virtual/block/dm-8 (block)
KERNEL[240932.696227] change   /devices/virtual/block/dm-8 (block)
UDEV  [240932.712517] change   /devices/virtual/block/dm-8 (block)
KERNEL[240932.793677] add      /devices/virtual/net/veth4d1712b (net)
KERNEL[240932.793699] add      /devices/virtual/net/veth4d1712b/queues/rx-0 (queues)
KERNEL[240932.793727] add      /devices/virtual/net/veth4d1712b/queues/tx-0 (queues)
KERNEL[240932.793908] add      /devices/virtual/net/veth17f6e3e (net)
KERNEL[240932.793930] add      /devices/virtual/net/veth17f6e3e/queues/rx-0 (queues)
KERNEL[240932.793942] add      /devices/virtual/net/veth17f6e3e/queues/tx-0 (queues)
UDEV  [240932.827598] add      /devices/virtual/net/veth4d1712b (net)
UDEV  [240932.827843] add      /devices/virtual/net/veth4d1712b/queues/rx-0 (queues)
UDEV  [240932.828121] add      /devices/virtual/net/veth4d1712b/queues/tx-0 (queues)
KERNEL[240932.849171] add      /kernel/slab/:A-0000200/cgroup/vm_area_struct(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.849196] add      /kernel/slab/:A-0000064/cgroup/anon_vma_chain(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.849211] add      /kernel/slab/:A-0000200/cgroup/vm_area_struct(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.849225] add      /kernel/slab/anon_vma/cgroup/anon_vma(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.849238] add      /kernel/slab/:aA-0000192/cgroup/dentry(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.849252] add      /kernel/slab/proc_inode_cache/cgroup/proc_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.849269] add      /kernel/slab/:A-0000064/cgroup/anon_vma_chain(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.849436] add      /kernel/slab/:0000064/cgroup/kmalloc-64(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.849455] add      /kernel/slab/shmem_inode_cache/cgroup/shmem_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.849709] add      /kernel/slab/proc_inode_cache/cgroup/proc_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.850033] add      /kernel/slab/anon_vma/cgroup/anon_vma(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.850374] add      /kernel/slab/shmem_inode_cache/cgroup/shmem_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.850527] add      /kernel/slab/:aA-0000192/cgroup/dentry(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.852368] add      /kernel/slab/:0000064/cgroup/kmalloc-64(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.854849] add      /kernel/slab/:0000192/cgroup/kmalloc-192(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.854871] add      /kernel/slab/:0001024/cgroup/kmalloc-1024(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855103] add      /kernel/slab/radix_tree_node/cgroup/radix_tree_node(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.855120] add      /kernel/slab/:0000192/cgroup/kmalloc-192(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855133] add      /kernel/slab/:A-0000192/cgroup/cred_jar(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855145] add      /kernel/slab/:A-0002048/cgroup/mm_struct(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855158] add      /kernel/slab/mqueue_inode_cache/cgroup/mqueue_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855170] add      /kernel/slab/sock_inode_cache/cgroup/sock_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855359] add      /kernel/slab/:A-0009664/cgroup/task_struct(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.855395] add      /kernel/slab/:A-0002048/cgroup/mm_struct(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855410] add      /kernel/slab/:A-0000704/cgroup/files_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.855445] add      /kernel/slab/mqueue_inode_cache/cgroup/mqueue_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855461] add      /kernel/slab/sighand_cache/cgroup/sighand_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.855474] add      /kernel/slab/:0001024/cgroup/kmalloc-1024(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855487] add      /kernel/slab/:A-0001024/cgroup/signal_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.855499] add      /kernel/slab/:A-0000128/cgroup/pid(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.855512] add      /kernel/slab/radix_tree_node/cgroup/radix_tree_node(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.856010] add      /kernel/slab/sock_inode_cache/cgroup/sock_inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.856029] add      /kernel/slab/:A-0009664/cgroup/task_struct(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.856045] add      /kernel/slab/sighand_cache/cgroup/sighand_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.856077] add      /kernel/slab/:0000256/cgroup/kmalloc-256(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.856092] add      /kernel/slab/:A-0000704/cgroup/files_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.856104] add      /kernel/slab/:0000512/cgroup/kmalloc-512(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.856188] add      /kernel/slab/:A-0000128/cgroup/pid(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.857080] add      /kernel/slab/:A-0000192/cgroup/cred_jar(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.857105] add      /kernel/slab/:A-0001024/cgroup/signal_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.857126] add      /kernel/slab/:0000256/cgroup/kmalloc-256(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.857140] add      /kernel/slab/:0000512/cgroup/kmalloc-512(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.862157] add      /kernel/slab/xfs_inode/cgroup/xfs_inode(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.862348] add      /kernel/slab/xfs_inode/cgroup/xfs_inode(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240932.863008] add      /kernel/slab/inode_cache/cgroup/inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.863179] add      /kernel/slab/inode_cache/cgroup/inode_cache(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240932.932564] add      /devices/virtual/net/veth17f6e3e (net)
UDEV  [240932.932779] add      /devices/virtual/net/veth17f6e3e/queues/rx-0 (queues)
UDEV  [240932.932880] add      /devices/virtual/net/veth17f6e3e/queues/tx-0 (queues)
KERNEL[240932.960183] remove   /devices/virtual/net/veth4d1712b (net)
UDEV  [240932.982851] remove   /devices/virtual/net/veth4d1712b (net)
KERNEL[240933.089679] add      /kernel/slab/:0000008/cgroup/kmalloc-8(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240933.089933] add      /kernel/slab/:0000008/cgroup/kmalloc-8(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240933.150833] add      /kernel/slab/:0002048/cgroup/kmalloc-2048(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
KERNEL[240933.150863] add      /kernel/slab/:0000096/cgroup/kmalloc-96(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240933.151052] add      /kernel/slab/:0002048/cgroup/kmalloc-2048(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)
UDEV  [240933.151130] add      /kernel/slab/:0000096/cgroup/kmalloc-96(990:53b4e439f50242461e4c1246718e12d769d3bc4752c0f0ed5bc4f317bb4feb3d) (cgroup)

```