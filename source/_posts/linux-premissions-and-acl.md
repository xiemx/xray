---
layout: post
title: Linux特殊权限和acl访问控制列表
published: true
author: xiemx
comments: true
date: 2015-11-05 09:11:28
tags: [ linux, acl ]
categories:
    - linux
---
### 特殊权限

linux中除了常见的读（r）、写（w）、执行（x）权限以外，还有3个特殊的权限，分别是setuid、setgid和stick bit。

#### setuid
只能设置文件，让普通用户拥有可以执行“**只有root权限才能执行**”的特殊权限，一般设置命令文件此属性。
设置方法：
```
chmod u+s /tmp/testfile
chmod u-s /tmp/testfile
```
如：
```
-rwsr-xr-x         1       root          root           22984          2015-11-04       /usr/bin/passwd
```
#### setgid
可设置文件和目录，当目录设置此属性时在目录下创建的文件，默认所属组都会变为该目录的所属组。

设置方法:
```
chmod g+s /tmp/test/
chmod g-s /tmp/test/
```
如:
```  
drwxr-sr-x         1       testuser         testadmin           22984          2015-11-04       /tmp/test/
-rw-r--r--           1       root         testadmin          22984          2015-11-04       /tmp/test/file1  ——文件所属组自动继承test的所属组
```
#### sticky bit
在目录下创建的文件，只有文件的拥有者和此目录的拥有者有编辑、删除权限

设置方法：
```
chmod +t /tmp
chmod -t /tmp
```
以上权限也可以通过数字的方式来设置特殊权限 setuid=4，setgid=2，sticky bit=1
```
chmod 4777 /test/file1 ——setuid
chmod 2777 /test/file2 ——setgid
chmod 1777 /test/      ——sticky bit
```

### attr权限（隐藏权限）

用来设置特殊用户例如：root之类的权限。


```
设置方法：
chattr  参数   文件

+ ：在原有参数设定基础上，追加参数。
- ：在原有参数设定基础上，移除参数。
= ：更新为指定参数设定。
A：文件或目录的 atime (access time)不可被修改(modified), 避免磁盘I/O读写占用资源。
S：硬盘I/O同步选项，功能类似sync。
a：即append，设定该参数后，只能向文件中添加数据，而不能删除，多用于服务器日志文件安全，只有root才能设定这个属性。
c：即compresse，设定文件是否经压缩后再存储。读取时需要经过自动解压操作。
d：即no dump，设定文件不能成为dump程序的备份目标。
i：设定文件不能被删除、改名、设定链接关系，同时不能写入或新增内容。i参数对于文件 系统的安全设置有很大帮助。
j：即journal，设定此参数使得当通过mount参数：data=ordered 或者 data=writeback 挂 载的文件系统，文件在写入时会先被记录(在journal中)。如果filesystem被设定参数为 data=journal，则该参数自动失效。
s：保密性地删除文件或目录，即文件的block、inode空间被全部清空收回。
u：与s相反，当设定为u时，数据内容其实还存在磁盘中，可以用于undeletion。
各参数选项中常用到的是a和i。a选项强制只可添加不可删除，多用于日志系统的安全设定。而i是更为严格的安全设定，只有superuser (root) 或具有CAP_LINUX_IMMUTABLE处理能力（标识）的进程能够施加该选项。
```
例：
```
chattr +i /tmp/testfile1  ——增加i权限
chattr +a /tmp/testfile1 ——增加a权限

查看文件的attr权限可使用lsattr命令
lsattr /tmp/testfile1   ——查看文件隐藏权限
```
#### ACL(访问控制列表)

Linux中`-rwxrwxrwx.`这种权限方式只能规范出三种类型的权限，在系统的使用中并不能完全的满足管理者对于用户权限的指定，因此Linux推出了ACL访问控制列表这种权限管理方法，此方法可以针对不同的用户和组来分配不同的权限配置不局限于原始的UGO权限类型。

ACL权限的种类
* R——读权限
* W——写权限
* X——执行权限

acl访问控制列表对于权限的规范和UGO类型的权限是一样的，也是通过RWX来组合控制用户权限。但设置方法不同具体如下

设置方法：
```
setfacl   参数    文件
-m   设置acl规则
-x   删除用户的acl规则
-b    清空acl规则
```
例：
```
setfacl  -m u:test:rwx /tmp/testfile1 ——赋予用户test对于文件testfile1有RWX权限
setfacl  -x u:test /tmp/testfile1     ——删除用户test对于文件testfile1的acl条目
setfacl  -b /tmp/testfile1            ——清空testfile1文件的acl条目
```
查看文件acl是否开启可以通过ls -l  查看文件权限字段 最后一位是否为+号。如：-rwxrwxrwx+则此文件开启了acl，也可通过命令getfacl  /tmp/testfile1来查看文件具体acl条目信息。