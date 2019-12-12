---
layout: post
title: URLRewriter设置Config和IIS配置做伪静态
published: true
author: xiemx
comments: true
date: 2015-08-21 10:08:51
tags: [ windows, iis]
categories:
    - 'iis'
    - 'windows'
#permalink: '/2015/08/21/iis-set-urlrewrite'
---

一、首先检查config文件里面是否包含这两个节点

```xml
 <configSections>

   <section name="RewriterConfig" requirePermission="false" type="URLRewriter.Config.RewriterConfigSerializerSectionHandler, URLRewriter"/>

 </configSections>

 <system.web>

   <httpModules>
     <add type="URLRewriter.ModuleRewriter, URLRewriter" name="ModuleRewriter"/>

   </httpModules>

 </system.web>

```

二、配置IIS。

 1.打开IIS，选择网站。右键，点击属性

 2.选择主目录，点击配置按钮

 3.在弹出的应用程序配置中，点击添加

 4.在弹出的添加/编辑应用程序扩展名映射中，填写可执行文件

​	点击浏览，选择aspnet_isapi.dll这个DLL，它一般在：C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727文件夹中(也就是你的系统盘中）

 5.在扩展名中填写.html

 6.在动作里选择限制为，他的框就可以填写了！请填写：GET,POST

 7.不要勾选把确认文件是否存在，然后点击确定

![image-20190917165314461](/images/image-20190917165314461.png)