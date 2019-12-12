---
layout: post
title: Tomcat虚拟主机别名设置
published: true
author: xiemx
comments: true
date: 2016-04-13 09:04:35
tags: [ tomcat, webserver, http ]
categories:
    - tomcat
# permalink: '/2016/04/13/tomcat%e8%99%9a%e6%8b%9f%e4%b8%bb%e6%9c%ba%e5%88%ab%e5%90%8d%e8%ae%be%e7%bd%ae'
---
一个虚拟空间需要绑定多个域名时可以通过alias标签来设置别名，详见如下配置文件部分截图
```xml
 <Engine name="Catalina" defaultHost="localhost">
      <!--For clustering, please take a look at documentation at:
          /docs/cluster-howto.html  (simple how to)
          /docs/config/cluster.html (reference documentation)
       -->
      <!--
      <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
      -->
      <!-- Use the LockOutRealm to prevent attempts to guess user passwords
           via a brute-force attack -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <!-- This Realm uses the UserDatabase configured in the global JNDI
             resources under the key "UserDatabase".  Any edits
             that are performed against this UserDatabase are immediately
             available for use by the Realm.  -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
        <Host name="www.xiemx.com" debug="0" appBase="webapps" unpackWARs="true" autoDeploy="true" xmlValidation="false" xmlNamespaceAware="false">
                <Alias>xiemx.com</Alias>
                <Alias>xx.xiemx.com</Alias>
                <Alias>yy.xiemx.com</Alias> 
        </Host>
  </Engine>
  ```