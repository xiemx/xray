---
layout: post
title: 网站支持.apk文件下载的设置方法
published: true
author: xiemx
comments: true
date: 2015-09-29 02:09:43
tags: [ iis, nginx, debug ]
categories:
    - iis
    - nginx
---
### iis支持.apk文件下载的设置

IIS服务器不能下载.apk文件的原因：iis的默认MIME类型中没有.apk文件，所以无法下载。

IIS服务器不能下载.apk文件的解决办法：既然.apk无法下载是因为没有MIME，那么添加一个MIME类型就可以了。

IIS服务器不能下载.apk文件的解决步骤：
```
打开IIS服务管理器，找到服务器，右键-属性，打开IIS服务属性；
单击MIME类型下的“MIME类型”按钮，打开MIME类型设置窗口；
单击“新建”，建立新的MIME类型；
扩展名中填写“.apk”,
MIME类型中填写apk的MIME类型`application/vnd.android.package-archive`

```
重启IIS，使设置生效。

### apache支持.apk文件下载的设置
在Apache安装目录下的`mime.types`加上以下一行语句
```
application/vnd.android.package-archive     apk;
```
重启apache即可

### nginx支持.apk文件下载的设置

apk 和 .ipa分别是android应用和ios应用的扩展名。

如果在浏览器下载这些文件为后缀的文件时，会自动重命名为zip文件。

当然可以下载后手动修改后缀，依然可以安装。

如果想下载后缀直接就是apk ipa的，可以修改 `/usr/local/nginx/conf`目录下的mime.types

```
application/vnd.android.package-archive apk;
application/iphone          pxl ipa;
```
增加上述配置，重启nginx生效
