---
layout: post
title: TCP状态机
published: true
author: xiemx
comments: true
date: 2018-01-23 11:01:20
tags: [ tcp, linux ]
categories:
    - linux
# permalink: '/2018/01/23/tcp-state'
---
### TCP状态分析
* listen／close
* syn-sent/syn-revd
* established
* fin_wait_1/close_wait
* fin_wait_2/last_ack
* time_wait/close

```
LISTEN	        等待来自远程TCP应用程序的请求
SYN_SENT	发送连接请求后等待来自远程端点的确认。TCP第一次握手后客户端所处的状态
SYN-RECEIVED	该端点已经接收到连接请求并发送确认。该端点正在等待最终确认。TCP第二次握手后服务端所处的状态
ESTABLISHED	代表连接已经建立起来了。这是连接数据传输阶段的正常状态
FIN_WAIT_1	等待来自远程TCP的终止连接请求或终止请求的确认
FIN_WAIT_2	在此端点发送终止连接请求后，等待来自远程TCP的连接终止请求
CLOSE_WAIT	该端点已经收到来自远程端点的关闭请求，此TCP正在等待本地应用程序的连接终止请求
CLOSING	        等待来自远程TCP的连接终止请求确认
LAST_ACK	等待先前发送到远程TCP的连接终止请求的确认
TIME_WAIT	等待足够的时间来确保远程TCP接收到其连接终止请求的确认
```

以上大致为一个Tcp从三次握手建立连接到四次挥手断开连接的整个过程C/S对应的TCP状态。
### 详细流程

```
1. 客户端(close)发送syn连接请求给服务端(listen),客户端等待服务端ack(syn_sent)
2. 服务端收到syn请求,发送ack/syn(syn_rec)
3. 客户端收到ack(establelished)
4. 传输数据
5. 客户端数据交互完成请求关闭连接，发送fin请求(fin_wait_1)
6. 服务端收到fin请求,发送ack(close_wait)
7. 服务端数据交互完成,发送fin请求关闭连接(last_ack)
8. 客户端收到服务端的ack请求(fin_wait_2)
9. 客户端收到服务端的fin请求,发送ack确认断开(time_wait)
10. 服务端收到客户端的ack,关闭连接(close)
11. 客户端维护2个msl时间后回收socket
```
引用网上的一张图：

![img](/images/img_5a66a577176c8.png)