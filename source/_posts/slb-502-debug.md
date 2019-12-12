---
layout: post
title: SLB 502报错Debug
published: true
author: xiemx
comments: true
date: 2018-07-20 02:07:06
tags: [ linux, nginx, debug ]
categories:
    - linux
    - nginx
    - debug
# permalink: '/2018/07/20/slb-502-debug'
---
用户自定义站点502问题分析

1. 现象：自定义域名用户反馈，打开网站返回502，如图

![img](/images/img_5b5183b84d4dd.png)

 根据response header判断，请求到达captain，怀疑captain返回的502页面。查看nginx proxy_pass得知后端的地址为bobcat.sxldns.com.

```
set $bobcat_backend "bobcat.sxldns.com";
proxy_pass http://$bobcat_backend;
```

使用curl模拟请求，直接请求bobcat.sxldns.com,正常获得返回内容，具体针对每个服务器的IP的curl,不再单独列出。

![img](/images/img_5b5184bfd49e3.png)

通过上述返回基本判断，问题出在我们的代理层。具体查看代理层的nginx配置和系统资源利用率。

查看当时系统的资源状态,查看到当时的系统磁盘空间使用完。查看nginx上有关于proxy的cache相关配置,nginx会cache的response的content的内容。怀疑nginx转发请求之后但是backend返回内容后，nginx cache到本地的时候无法写入disk导致会话结束，SLB的请求无返回包认为后端宕机抛出502。

```
proxy_cache_path /etc/nginx/china_cache levels=1:2 keys_zone=user_page_cache:100m max_size=20g inactive=60m use_temp_path=off;
```

具体DEBUG如下：

 1. 获取container中的nginx worker进程的PID

![img](/images/img_5b5184fb1c8d4.png)

2. strace查看下系统调用具体信息

![img](/images/img_5b518515e4fe7.png)

通过上图strace追踪流程可以看到左侧是一个正常的请求的全部流程，右侧是故障状态的strace的系统调用全流程。

通过对比左右两侧的系统调用流程可以看到，当nginx cache的时候写入`/etc/nginx/china_cache`目录时提示`no space`后续的writev()和sendfile()方法就没有调用，因此导致SLB无法获得返回包，抛出`badgateway`的错误.

PS：

1. 为什么单独请求返回头的时候正常返回？nginx返回请求头的时候并不会走proxy的cache流程。因此没有调用open()方法读写disk，正常返回，但是实际请求数据的时候cache写磁盘直接失败，后续直接退出。
2. 上图的系统调用过程为了debug的方便使用了非折叠模式。因此一些no space的一些报错未显示出来可以。全部流程见附录。

附录：

```
1.异常调用堆栈
[root@iZ2ze2mzhjk3ou1vsnkthzZ rpm]# strace -p 3904 -v
strace: Process 3904 attached
epoll_wait(8, [{EPOLLIN, {u32=69025808, u64=140003117973520}}], 512, -1) = 1
accept4(6, {sa_family=AF_INET, sin_port=htons(59464), sin_addr=inet_addr("10.130.0.4")}, [16], SOCK_NONBLOCK) = 11
epoll_ctl(8, EPOLL_CTL_ADD, 11, {EPOLLIN|EPOLLRDHUP|EPOLLET, {u32=69027249, u64=140003117974961}}) = 0
epoll_wait(8, [{EPOLLIN, {u32=69027249, u64=140003117974961}}], 512, 60000) = 1
recvfrom(11, "GET /?key=testxxxx HTTP/1.1\r\nUse"..., 1024, 0, NULL, NULL) = 88
epoll_ctl(8, EPOLL_CTL_MOD, 11, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=69027249, u64=140003117974961}}) = 0
open("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2", O_RDONLY|O_NONBLOCK) = 14
fstat(14, {st_dev=makedev(253, 1), st_ino=655502, st_mode=S_IFREG|0600, st_nlink=1, st_uid=101, st_gid=101, st_blksize=4096, st_blocks=96, st_size=46750, st_atime=2018/07/17-16:34:43.967698739, st_mtime=2018/07/17-16:34:43.966698736, st_ctime=2018/07/17-16:34:43.967698739}) = 0
pread64(14, "\5\0\0\0\0\0\0\0(\252M[\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 636, 0) = 636
getsockname(11, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("172.17.0.2")}, [16]) = 0
sendto(13, "nN\1\0\0\1\0\0\0\0\0\0\6bobcat\6sxldns\3com\0\0"..., 35, 0, NULL, 0) = 35
sendto(13, "\375\215\1\0\0\1\0\0\0\0\0\0\6bobcat\6sxldns\3com\0\0"..., 35, 0, NULL, 0) = 35
epoll_wait(8, [{EPOLLOUT, {u32=69027249, u64=140003117974961}}, {EPOLLIN, {u32=69026288, u64=140003117974000}}], 512, 5000) = 2
recvfrom(13, "nN\201\200\0\1\0\3\0\0\0\0\6bobcat\6sxldns\3com\0\0"..., 4096, 0, NULL, NULL) = 147
recvfrom(13, "\375\215\201\200\0\1\0\1\0\1\0\0\6bobcat\6sxldns\3com\0\0"..., 4096, 0, NULL, NULL) = 168
socket(AF_INET, SOCK_STREAM, IPPROTO_IP) = 15
ioctl(15, FIONBIO, [1]) = 0
epoll_ctl(8, EPOLL_CTL_ADD, 15, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=69027488, u64=140003117975200}}) = 0
connect(15, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("54.222.148.216")}, 16) = -1 EINPROGRESS (Operation now in progress)
recvfrom(13, 0x7ffd1f603790, 4096, 0, NULL, NULL) = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(8, [{EPOLLOUT, {u32=69027488, u64=140003117975200}}], 512, 20000) = 1
getsockopt(15, SOL_SOCKET, SO_ERROR, [0], [4]) = 0
writev(15, [{"GET /?key=testxxxx HTTP/1.0\r\nHos"..., 219}], 1) = 219
epoll_wait(8, [{EPOLLIN|EPOLLOUT, {u32=69027488, u64=140003117975200}}], 512, 60000) = 1
recvfrom(15, "HTTP/1.1 200 OK\r\nContent-Type: t"..., 3723, 0, NULL, NULL) = 3723
close(14) = 0
readv(15, [{"a.qnssl.com/images/265818/Fntncr"..., 4096}], 1) = 4096
readv(15, [{"w-card-price{color: #004aa0;}.s-"..., 4096}], 1) = 4096
readv(15, [{" {\n font-family: \"Open Sans"..., 4096}], 1) = 373
open("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2.0000014052", O_RDWR|O_CREAT|O_EXCL, 0600) = 14
pwritev(14, [{"\5\0\0\0\0\0\0\0\356\254M[\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 4096}, {"a.qnssl.com/images/265818/Fntncr"..., 4096}, {"w-card-price{color: #004aa0;}.s-"..., 4096}], 3, 0) = -1 ENOSPC (No space left on device)
gettid() = 7
write(4, "2018/07/17 08:46:33 [crit] 7#7: "..., 328) = 328
close(15) = 0
unlink("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2.0000014052") = 0
write(5, "{ \"time\": \"2018-07-17T08:46:33+0"..., 382) = 382
close(14) = 0
close(11) = 0

2.正常调用堆栈过程
[root@iZ2ze2mzhjk3ou1vsnkthzZ rpm]# strace -p 3904 -v
strace: Process 3904 attached
epoll_wait(8, [{EPOLLIN, {u32=69025808, u64=140003117973520}}], 512, -1) = 1
accept4(6, {sa_family=AF_INET, sin_port=htons(59360), sin_addr=inet_addr("10.130.0.4")}, [16], SOCK_NONBLOCK) = 11
epoll_ctl(8, EPOLL_CTL_ADD, 11, {EPOLLIN|EPOLLRDHUP|EPOLLET, {u32=69027248, u64=140003117974960}}) = 0
epoll_wait(8, [{EPOLLIN, {u32=69027248, u64=140003117974960}}], 512, 60000) = 1
recvfrom(11, "GET /?key=testxxxx HTTP/1.1\r\nUse"..., 1024, 0, NULL, NULL) = 88
epoll_ctl(8, EPOLL_CTL_MOD, 11, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=69027248, u64=140003117974960}}) = 0
open("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2", O_RDONLY|O_NONBLOCK) = 14
fstat(14, {st_dev=makedev(253, 1), st_ino=655504, st_mode=S_IFREG|0600, st_nlink=1, st_uid=101, st_gid=101, st_blksize=4096, st_blocks=96, st_size=46750, st_atime=2018/07/17-16:14:38.127407158, st_mtime=2018/07/17-16:14:38.127407158, st_ctime=2018/07/17-16:14:38.128407161}) = 0
pread64(14, "\5\0\0\0\0\0\0\0s\245M[\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 636, 0) = 636
getsockname(11, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("172.17.0.2")}, [16]) = 0
sendto(12, "\210V\1\0\0\1\0\0\0\0\0\0\6bobcat\6sxldns\3com\0\0"..., 35, 0, NULL, 0) = 35
sendto(12, "\27\1\1\0\0\1\0\0\0\0\0\0\6bobcat\6sxldns\3com\0\0"..., 35, 0, NULL, 0) = 35
epoll_wait(8, [{EPOLLOUT, {u32=69027248, u64=140003117974960}}, {EPOLLIN, {u32=69027728, u64=140003117975440}}], 512, 5000) = 2
recvfrom(12, "\210V\201\200\0\1\0\3\0\0\0\0\6bobcat\6sxldns\3com\0\0"..., 4096, 0, NULL, NULL) = 147
recvfrom(12, "\27\1\201\200\0\1\0\1\0\1\0\0\6bobcat\6sxldns\3com\0\0"..., 4096, 0, NULL, NULL) = 168
socket(AF_INET, SOCK_STREAM, IPPROTO_IP) = 15
ioctl(15, FIONBIO, [1]) = 0
epoll_ctl(8, EPOLL_CTL_ADD, 15, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=69027489, u64=140003117975201}}) = 0
connect(15, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("52.80.58.42")}, 16) = -1 EINPROGRESS (Operation now in progress)
recvfrom(12, 0x7ffd1f603790, 4096, 0, NULL, NULL) = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(8, [{EPOLLOUT, {u32=69027489, u64=140003117975201}}], 512, 20000) = 1
getsockopt(15, SOL_SOCKET, SO_ERROR, [0], [4]) = 0
writev(15, [{"GET /?key=testxxxx HTTP/1.0\r\nHos"..., 219}], 1) = 219
epoll_wait(8, [{EPOLLIN|EPOLLOUT, {u32=69027489, u64=140003117975201}}], 512, 60000) = 1
recvfrom(15, "HTTP/1.1 200 OK\r\nContent-Type: t"..., 3723, 0, NULL, NULL) = 3723
close(14) = 0
readv(15, [{"a.qnssl.com/images/265818/Fntncr"..., 4096}], 1) = 4096
readv(15, [{"w-card-price{color: #004aa0;}.s-"..., 4096}], 1) = 4096
readv(15, [{" {\n font-family: \"Open Sans"..., 4096}], 1) = 2565
open("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2.0000014051", O_RDWR|O_CREAT|O_EXCL, 0600) = 14
pwritev(14, [{"\5\0\0\0\0\0\0\0(\252M[\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 4096}, {"a.qnssl.com/images/265818/Fntncr"..., 4096}, {"w-card-price{color: #004aa0;}.s-"..., 4096}], 3, 0) = 12288
writev(11, [{"HTTP/1.1 200 OK\r\nServer: nginx/1"..., 321}], 1) = 321
sendfile(11, 14, [636] => [12288], 11652) = 11652
epoll_wait(8, [{EPOLLOUT, {u32=69027489, u64=140003117975201}}], 512, 59955) = 1
epoll_wait(8, [{EPOLLIN|EPOLLOUT, {u32=69027489, u64=140003117975201}}], 512, 59953) = 1
readv(15, [{"rc=\"//nzr2ybsda.qnssl.com/images"..., 1531}, {"lass=\"s-nav-item\" target=\"_self\""..., 4096}, {"lign: left; font-size: 160%;\">\302\240"..., 4096}, {"OQswmpPvoA0.jpg?imageMogr2/strip"..., 4096}], 4) = 10136
pwritev(14, [{" {\n font-family: \"Open Sans"..., 4096}, {"lass=\"s-nav-item\" target=\"_self\""..., 4096}, {"lign: left; font-size: 160%;\">\302\240"..., 4096}], 3, 12288) = 12288
sendfile(11, 14, [12288] => [24576], 12288) = 12288
epoll_wait(8, [{EPOLLIN|EPOLLOUT, {u32=69027489, u64=140003117975201}}], 512, 59948) = 1
readv(15, [{"olor-custom1\">\347\254\254\344\270\211\346\226\271\345"..., 3683}, {"container title-group-container\""..., 4096}, {";cs=srgb&s=f71a6adc1e5953a52"..., 4096}, {")\" data-bg=\"//nzr2ybsda.qnssl.co"..., 4096}], 4) = 15971
readv(15, [{"\230\263\345\224\257\346\231\237\" title=\"\346\262\210\351\230\263\345\224\257\346\231\237\345\225\206"..., 4096}], 1) = 2853
pwritev(14, [{"OQswmpPvoA0.jpg?imageMogr2/strip"..., 4096}, {"container title-group-container\""..., 4096}, {";cs=srgb&s=f71a6adc1e5953a52"..., 4096}, {")\" data-bg=\"//nzr2ybsda.qnssl.co"..., 4096}], 4, 24576) = 16384
sendfile(11, 14, [24576] => [40960], 16384) = 16384
epoll_wait(8, [{EPOLLIN|EPOLLOUT, {u32=69027489, u64=140003117975201}}], 512, 59947) = 1
readv(15, [{"

<d"..., 1243}, {"\232\204\344\270\232\347\273\251\344\271\237\345\234\250\344\270\215\346\226\255\346\224\200\345\215\207\343\200\202</d"..., 4096}, {"", 4096}, {"", 4096}, {"", 4096}], 5) = 2937
pwritev(14, [{"\230\263\345\224\257\346\231\237\" title=\"\346\262\210\351\230\263\345\224\257\346\231\237\345\225\206"..., 4096}, {"\232\204\344\270\232\347\273\251\344\271\237\345\234\250\344\270\215\346\226\255\346\224\200\345\215\207\343\200\202</d"..., 1694}], 2, 40960) = 5790 sendfile(11, 14, [40960] => [46750], 5790) = 5790
chmod("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2.0000014051", 0600) = 0
rename("/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2.0000014051", "/etc/nginx/china_cache/2/7d/2ac65d1f9080e2baff45a4332f9017d2") = 0
fstat(14, {st_dev=makedev(253, 1), st_ino=655502, st_mode=S_IFREG|0600, st_nlink=1, st_uid=101, st_gid=101, st_blksize=4096, st_blocks=96, st_size=46750, st_atime=2018/07/17-16:34:43.967698739, st_mtime=2018/07/17-16:34:43.966698736, st_ctime=2018/07/17-16:34:43.967698739}) = 0
close(15) = 0
write(5, "{ \"time\": \"2018-07-17T08:34:43+0"..., 386) = 386
close(14) = 0
setsockopt(11, SOL_TCP, TCP_NODELAY, [1], 4) = 0
epoll_wait(8, [{EPOLLIN|EPOLLOUT|EPOLLRDHUP, {u32=69027248, u64=140003117974960}}], 512, 65000) = 1
recvfrom(11, "", 1024, 0, NULL, NULL) = 0
close(11) = 0
```