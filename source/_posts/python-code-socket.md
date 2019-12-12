---
layout: post
title: python socket编程
published: true
author: xiemx
comments: true
date: 2016-04-28 02:04:14
tags: [ python, socket ]
categories:
    - python
# permalink: '/2016/04/28/socket%e7%bc%96%e7%a8%8b'
---
需要在多台服务器上运行日志分析脚本，分析完成后每台机器直接发送邮件会出现大量邮件同时过来，现在想将多台机器的分析结果收集起来，通过网络发送到服务端，在服务端收集所有的日志，在统一发送一封邮件，实现的socket代码如下。

### 接收端（server）
```python
#!/usr/bin/env python
import socket
def socket_server(host,port):
    self_host = host
    self_port = port

    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.bind((self_host,self_port))
    s.listen(4)

    f = open("log", "a")
    while True:
        conn,addr=s.accept()
        data = conn.recv(1024)
        print 'data:', data
        f.write(data)
    f.close()
    s.close()

if __name__ == "__main__":
    host = '127.0.0.1'
    port = 65530

    socket_server(host,port)
```
### 发送端（client）
```python
#!/usr/bin/env python
import socket

def socket_client(host,port,content):
    self_host = host
    self_port = port
    self_content = content

    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((self_host,self_port))
    s.sendall(content)
    s.close()

if __name__ == "__main__":
    host = "127.0.0.1"
    port = 65530

    f = open("sendfile","r")
    content = f.read()
    f.close()

    socket_client(host, port, content)
```