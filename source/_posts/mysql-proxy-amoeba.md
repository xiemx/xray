---
layout: post
title: Mysql数据库代理amoeba
published: true
author: xiemx
comments: true
date: 2016-02-18 05:02:36
tags: [ mysql, amoeba, mysql-proxy]
categories:
    - mysql
---
基本环境：
```
A（172.25.16.10）：客户端    
B（172.25.16.11）：主数据库       
C（172.25.16.12）：从数据库     
D（172.25.16.13）：从数据库
J（172.25.16.19）：代理数据库          
```

基于amoeba的读写分离

1. 在server j 安装JDK

2. 新建目录/usr/local/amoeba
```shell
mkdir /usr/local/amoeba
```
3. 把压缩包解压到该目录/usrl/local/amoeba

4. 在conf目录下有amoeba的配置文件
  需要修改的两个：`amoeba.xml` 和 `dbServers.xml`
### amoeba.xml(3个地方要改):
#### 监听的端口号：
```
<proxy>
<!-- service class must implements com.meidusa.amoeba.service.Service -->
  <service name="Amoeba for Mysql" class="com.meidusa.amoeba.net.ServerableConnectionManager">
<!-- port -->
<property name="port">3306</property>
```
#### 客户端访问时用的用户名
```
<property name="authenticator">

  <bean class="com.meidusa.amoeba.mysql.server.MysqlClientAuthenticator">
  <property name="user">amoeba</property>
  <property name="password">amoeba</property>
  <property name="filter">
    <bean class="com.meidusa.amoeba.server.IPAccessController">
      <property name="ipFile">${amoeba.home}/conf/access_list.conf</property>
    </bean>
  </property>
  </bean>
</property>
```
#### read/write pool
```
<property name="sqlFunctionFile">${amoeba.home}/conf/functionMap.xml</property>
<property name="LRUMapSize">1500</property>
     <property name="defaultPool">serverb#默认访问的数据库#</property>
            <property name="writePool">serverb#指定可写池#</property>
            <property name="readPool">readgroup1#指定只读池#</property>
<property name="needParse">true</property>
```

### dbServers.xml文件：

#### 修改端口号，用户名，密码。
```
<dbServer name="abstractServer" abstractive="true">
<factoryConfig class="com.meidusa.amoeba.mysql.net.MysqlServerConnectionFactory">
  <property name="manager">${defaultManager}</property>
  <property name="sendBufferSize">64</property>
  <property name="receiveBufferSize">128</property>
  <!-- mysql port -->
  <property name="port">3306</property>
  <!-- mysql schema -->
  <property name="schema">test</property>
  <!-- mysql user -->
  <property name="user">dbproxy</property>
  <!--  mysql password  -->
  <property name="password">xiemx</property>
</factoryConfig>
```
 

#### 指定可以代理的数据库
```
<dbServer name="serverb"  parent="abstractServer">
  <factoryConfig>
    <!-- mysql ip -->
    <property name="ipAddress">172.25.16.11</property>
  </factoryConfig>
</dbServer>

<dbServer name="serverc"  parent="abstractServer">
  <factoryConfig>
    <!-- mysql ip -->
    <property name="ipAddress">172.25.16.12</property>
  </factoryConfig>
</dbServer>

<dbServer name="serverd"  parent="abstractServer">
  <factoryConfig>
    <!-- mysql ip -->
    <property name="ipAddress">172.25.16.13</property>
  </factoryConfig>
</dbServer>
```
 

#### 指定只读组
```
<dbServer name="readgroup1" virtual="true">
  <poolConfig class="com.meidusa.amoeba.server.MultipleServerPool">
    <!-- Load balancing strategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->
    <property name="loadbalance">1</property>
    <!-- Separated by commas,such as: server1,server2,server1 -->
    <property name="poolNames">serverc,serverd</property>
  </poolConfig>
</dbServer>
```
 

5. 启动脚本在/bin下
启动时会报错，需要修改文件即可：`DEFAULT_OPTS="-server -Xms256m -Xmx256m -Xss228k"`(最后一个参数改为228即可)
```shell
./amoeba start
```
6. 在BCD上授权允许J访问数据库
```sql
grant all on db1.* to dbproxy@'172.25.16.19' identified by 'xiemx';
```