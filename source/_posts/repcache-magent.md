---
layout: post
title: repcache＋magent构建高可用memcache
published: true
author: xiemx
comments: true
date: 2017-02-09 06:02:53
tags: [ memcached, repcache, magent]
categories:
    - memcached
# permalink: '/2017/02/09/repcache-magent'
---
repcache+magnet

magent：一款开源的Memcached代理服务器软件，功能和mysql-proxy类似。
  
repcache：日本开发的一款开源工具，使memcache能够做主从复制。可以通过patch包升级memcache，也可以下载包含replication的memcache版本

先将master/slave通过replication构建自动复制的主主，通过magent将K／V，写入到master和backup中，当master宕机时，magent将所有读写请求交给slave，slave等待master启动，默认情况下master宕机重启后内存中的数据回丢失，但由于repcache的自动主主，master启动时会自动从salve端复制所有数据。

### 安装部署
```shell
libevent:
  apt-get install -y libevent-dev

magent：
  wget http://memagent.googlecode.com/files/magent-0.5.tar.gz  
  tar zxvf magent-0.5.tar.gz  
  /sbin/ldconfig  
  sed -i "s#LIBS = -levent#LIBS = -levent -lm#g" Makefile  
  make  
  cp magent /usr/bin/magent  
  
repcache:
  wget http://downloads.sourceforge.net/repcached/memcached-1.2.8-repcached-2.2.tar.gz
  tar xf memcached-1.2.8-repcached-2.2.tar.gz
  cd memcached-1.2.8-repcached-2.2/
  ./configure --prefix=/opt/repcached/ --enable-replication --program-transform-name=s/memcached/repcached/
  make&&make install
  ln -s /opt/repcached/bin/repcached /usr/bin/
```

### 编译错误汇总
```shell
### magent

[root@test magent]# make 
gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o magent.o magent.c
magent.c: In function ‘writev_list’:
magent.c:729: error: ‘SSIZE_MAX’ undeclared (first use in this function)
magent.c:729: error: (Each undeclared identifier is reported only once
magent.c:729: error: for each function it appears in.)
make: *** [magent.o] Error 1
解决方法:
[root@test magent]# vim ketama.h
#ifndef SSIZE_MAX
#define SSIZE_MAX      32767
#endif
#ifndef _KETAMA_H
#define _KETAMA_H

[root@test magent]# make 
gcc -Wall -O2 -g  -c -o magent.o magent.c
gcc -Wall -O2 -g  -c -o ketama.o ketama.c
gcc -Wall -O2 -g -o magent magent.o ketama.o -levent
ketama.o: In function `create_ketama':
/opt/root/magent-0.5/ketama.c:399: undefined reference to `floorf'
collect2: ld returned 1 exit status
make: *** [magent] Error 1

修改Makefile
LIBS = -levent 为LIBS = -levent -lm
#sed -i "s#LIBS = -levent#LIBS = -levent -lm#g" Makefile 
```
```shell
### repcached

[root@test magent]# make 
make all-recursive    
make[1]: Entering directory `/usr/local/memcached'    
Making all in doc    
make[2]: Entering directory `/usr/local/memcached/doc'    
make[2]: Nothing to be done for `all'.    
make[2]: Leaving directory `/usr/local/memcached/doc'    
make[2]: Entering directory `/usr/local/memcached'    
gcc -DHAVE_CONFIG_H -I. -DNDEBUG -m64 -g -O2 -MT memcached-memcached.o -MD     
MP -MF .deps/memcached-memcached.Tpo -c -o memcached-memcached.o `test -f     
memcached.c' || echo './'`memcached.c    
memcached.c: In function ‘add_iov’:    
memcached.c:697: error: ‘IOV_MAX’ undeclared (first use in this function)    
memcached.c:697: error: (Each undeclared identifier is reported only once    
memcached.c:697: error: for each function it appears in.)    
make[2]: *** [memcached-memcached.o] Error 1    
make[2]: Leaving directory `/usr/local/memcached'    
make[1]: *** [all-recursive] Error 1    
make[1]: Leaving directory `/usr/local/memcached'    
make: *** [all] Error 2
    
解决方案：    
vi memcached.c
    
将下面的几行    
/* FreeBSD 4.x doesn't have IOV_MAX exposed. */    
#ifndef IOV_MAX    
#if defined(__FreeBSD__) || defined(__APPLE__)    
# define IOV_MAX 1024    
#endif    
#endif
    
修改为
/* FreeBSD 4.x doesn't have IOV_MAX exposed. */    
#ifndef IOV_MAX    
# define IOV_MAX 1024    
#endif
```
### 启动服务
```shell
### 启动repcached
启动master：
repcached -d -v -x 127.0.0.1 -u vagrant  
启动slave：
repcached -d -v -x 127.0.0.1 -u vagrant -p 11222

参数说明：
  -x 设置从哪个IP上进行同步。
  -X 指定数据同步的端口。
  Memcached默认服务端口是11211，默认同步监听端口是11212。

### 启动magent
magent -u root -p 11233 -s 127.0.0.1:11211 -s 127.0.0.1:11222
```
