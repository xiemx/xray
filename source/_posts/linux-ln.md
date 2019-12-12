---
layout: post
title: Linux链接文件（ln）
published: true
author: xiemx
comments: true
date: 2015-11-05 09:11:15
tags: [ linux, command ]
categories:
    - linux
---
Linux系统中,内核为每一个文件分配一个`inode` ，文件属性保存在inode里，在访问文件时，文件系统通过查找到文件的inode中关于文件的存放block，进而去相对应的block中读取文件内容。而链接是一种在共享文件和访问它的用户的若干目录项之间建立联系的一种方法，在系统中有相同的一份文件A和B两个用户都需要这个文件，传统模式下我们会直接复制文件分给2个用户，这样虽然能实现目的。但是两份文件占用了我们双倍的block和inode资源。如果这个文件非常大的话将造成资源的极大浪费。此时Linux引入链接的概念，我们可以通过链接的方式去将文件共享给两个用户。

### 两种链接：
* 硬链接(Hard Link)
* 和软链接(Soft Link)，软链接又称为符号链接（Symbolic link）

#### 硬链接

硬链接文件可以理解为一个指针，指向文件的inode，系统不重新分配inode。硬链接文件中记录的是源文件的inode信息，所以硬链接文件的属性和源文件的属性相同只是在硬链接连接数上+1，硬链接文件。

  创建方法：
```
  ln    源文件    目标文件
```
  例子：
```
  ln   /tmp/testdir/testdir2/testfile1   /tmp/testfile1
  ls  -i   /tmp/testdir/testdir2/testfile1   /tmp/testfile1
  1491138      -rw-r–r–       1         root     root     48        11-14 14:10       /tmp/testdir/testdir2/testfile1
  1491138      -rw-r–r–       1         root     root     48        11-14 14:29       /tmp/testfile1
```

  注意事项：
  1.硬链接文件无法跨文件系统，因为不同的文件系统inode信息记录的信息不同。
  2.只能对文件设置硬链接
  3.删除硬链接文件或源文件，只会使硬链接计数器-1，不会删除文件内容。直到硬链接计数器为1时，在删除文件，此时则会清除文件。
  4.修改一个文件，其他文件同时被修改

#### 软连接

软连接通过存储源文件的路径来定位源文件。所以软连接文件的大小为路径的长度。软链接文件属性同源文件属性不同。

  创建方法:
```
  ln    -s      源文件  目标文件
```
  例子：
```
  ln  -s    /tmp/testfile    /testfilesoft
  ls   -i  /tmp/testfile   /testfilesoft
  1491138      -rw-r–r–       1         root     root     48        11-14 14:17        /tmp/testfile 1491140      lrwxrwxrwx  1         root     root     13          11-14 14:24        /testfilesoft -> /tmp/testfile
```
  查看以上文件的输出结果，源文件和链接文件的属性不同。链接文件的大小为13字节，这13字节等于软连接的路径“/tmp/testfile"的长度。

  注意事项：

  1.软连接文件记录的是文件路径因此可以跨文件系统创建
  2.可针对目录设置软连接
  3.删除软连接文件不影响其它链接文件的访问，删除源文件则所有软连接文件都无法访问。
  4.修改任意文件，所有文件都会变动

Linux系统中软硬链接的方式可使用的环境不同，其中软连接使用的最多