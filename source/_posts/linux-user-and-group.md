---
layout: post
title: Linux用户和组
published: true
author: xiemx
comments: true
date: 2015-11-05 07:11:13
tags: [ linux, commands]
categories:
    - linux
---
### 用户和组的配置文件
在linux中，用户帐号，用户密码，用户组信息和用户组密码均是存放在不同的配置文件中的。
* `/etc/passwd`存放用户配置信息
* `/etc/shadow`存放用户密码和密码策略
* `/etc/group`存放组配置信息
* `/etc/gshadow`存放组配置信息


#### /etc/passwd文件中每行存储一个用户信息以：分割分7段具体含义如下
* **用户帐号:密码占位符:用户ID:用户组ID:描述:用户主目录:用户所使用的shell**

```shell
[mingxu.xie@cn-aux-cc ~]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```
#### /etc/shadow文件对应passwd文件分9段，具体信息如下
* **用户名:密码:最近修改时间:最短有效时间:最长有效时间:过期提醒日期:过期后宽限时间:账户失效时间:保留字段**
```shell
[mingxu.xie@cn-aux-cc ~]$ sudo cat /etc/shadow
root:*LOCK*:14600::::::
bin:*:16323:0:99999:7:::
daemon:*:16323:0:99999:7:::
adm:*:16323:0:99999:7:::
lp:*:16323:0:99999:7:::
test:$6$getqHfvX$xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx:18185:0:99999:7:::
```

#### /etc/group文件保存了所有组的信息。文件中每一行表示一个组由3个冒号分隔成4段，具体信息如下

* **组名:密码占位符:组ID:组成员**
```shell
[mingxu.xie@cn-aux-cc ~]$ sudo cat /etc/group
root:x:0:
bin:x:1:bin,daemon
daemon:x:2:bin,daemon
sys:x:3:bin,adm
adm:x:4:adm,daemon
tty:x:5:
disk:x:6:
lp:x:7:daemon
mem:x:8:
kmem:x:9:
wheel:x:10:ec2-user,test
```

#### /etc/gshadow文件对应group文件，分4段，具体信息如下

* **组名:口令:组管理者:组内用户列表**
```shell
[mingxu.xie@cn-aux-cc ~]$ sudo cat /etc/gshadow
root:::
bin:::bin,daemon
daemon:::bin,daemon
sys:::bin,adm
adm:::adm,daemon
```
**修改以上文件可以完成对用户和组属性和配置的改变，/sbin/nologin的shell可以使用户不可登陆等等。也可以通过命令来修改用户属性，但实质也是修改此配置文件中的参数。**

### 用户和组的操作
```shell
1、添加新用户
  useradd 选项 用户名 

  其中各选项含义如下： 
  -c comment 指定一段注释性描述。
  -d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，能创建主目录。 
  -g 指定用户所属的用户组。 
  -G 指定用户所属的附加组。 
  -s  指定用户的登录Shell。 
  -u 指定用户的用户号，如果同时有-o选项，则能重复使用其他用户的标识号。

useradd  -G bin -g root -s /sbin/noloing -u 101   testuser

2、添加组
  用法：groupadd    选项    用户名 
  其中各选项含义如下： 
  -g 指定用户组ID

groupadd  -g 40000  testgrp

3、修改用户属性和组属性
  groupmod  ——修改组属性
  usermod     ——修改用户属性
  参数、语法和创建时使用的相同。

4、删除用户和组
  userdel testuser   ——默认删除testuser用户的配置但不删除家目录和邮件需手工删除
  userdel -r testuser  ——删除用户同时删除家目录文件和邮件
  groupdel testgrp  ——删除组

5、修改用户密码策略

[mingxu.xie@cn-aux-cc ~]$ chage --help
Usage: chage [options] [LOGIN]

Options:
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -h, --help                    display this help message and exit
  -I, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -l, --list                    show account aging information
  -m, --mindays MIN_DAYS        set minimum number of days before password
                                change to MIN_DAYS
  -M, --maxdays MAX_DAYS        set maximim number of days before password
                                change to MAX_DAYS
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
系统添加用户时如果没有规定用户的详细属性，则系统默认参照/etc/login.defs  和/etc/default/useradd来配置用户的属性！修改此处文档可以设置默认用户添加时的属性。

```