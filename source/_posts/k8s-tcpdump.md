---
layout: post
title: 如何在K8S环境中抓POD的包
published: true
author: xiemx
comments: true
date: 2019-12-12 10:01:04
tags: [ k8s, tcpdump, linux, network]
categories:
    - k8s
---
### 如何在K8S环境中抓POD的包

1. kubectl get pod -o wide 获取pod所在的node信息
```shell
➜  Documents kubectl get pod -o wide 
NAME                                                 READY   STATUS             RESTARTS   AGE   IP             NODE                                              NOMINATED NODE
internal-nginx-ingress-controller-7fdf7f457d-bd59z   1/1     Running            0          42m   10.200.1.83    ip-10-200-1-202.ap-northeast-1.compute.internal   <none>

2. kubectl describe pod/podname 获取pod的containid
```shell
➜  Documents k get pod -o jsonpath='{.status.containerStatuses[*].containerID}' internal-nginx-ingress-controller-7fdf7f457d-bd59z
docker://ae9a6df60584e797e56cc64d0df02e64d7731a0d852026fab0a76c920c608cbe
```

3. 登陆node节点，找到container查看eth0网卡的ID
```shell
[ec2-user@ip-10-200-1-202 net]$ docker exec -it ae9a6df60584e797e56cc64d0df02e64d7731a0d852026fab0a76c920c608cbe cat /sys/class/net/eth0/iflink
88
```

4. 宿主机上查询对应ID的网卡设备号
```
[ec2-user@ip-10-200-1-202 net]$ cd /sys/class/net; for i in $(ls);do echo $i ;grep 88 $i/ifindex;done
eni0143b083c86
eni154a5470c40
eni1c162323f07
eni1d3e2ba2ce1
eni3fbceb3330b
eni457702aeb41
eni45f360a240e
eni50431e3a94f
eni619e29d4bac
eni66339821adf
eni6fe679d6356
eni79708a78f8b
eni7cc26b0b7d2
eni855ca0ba49b
eni8799376f27c
eni90208382a7b
eni909411bbf11
eni94c3d2bb833
enia14f5f7c3e9
enib70b44b2399
enic2ad9523b38 ###容器所属网卡
88
enid60e48c6616
enid8f13b5dd06
enida858799e91
enieee4f7696a1
enif0d5e81d420
eth0
eth1
lo
```

5. tcpdump 抓包即可
```shell
➜  ssh-keys git:(master) ✗ ssh -F ~/.matrix/jp/ssh.aux.config 10.200.1.202 -l ec2-user "sudo tcpdump -vvv -i enic2ad9523b38 tcp port 80 -w -" | wireshark -k -i -
Warning: Permanently added '10.200.1.4' (ECDSA) to the list of known hosts.
Warning: Permanently added '10.200.1.202' (ECDSA) to the list of known hosts.
tcpdump: listening on eni86f5b593a42, link-type EN10MB (Ethernet), capture size 262144 bytes
tcpdump: pcap_loop: The interface went down
7408 packets captured
7408 packets received by filter
0 packets dropped by kernel
```