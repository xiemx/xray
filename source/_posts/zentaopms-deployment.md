---
layout: post
title: Zentaopms部署
published: true
author: xiemx
comments: true
date: 2016-03-25 11:03:57
tags: [ zentaopms, deployment]
categories:
    - 'zentaopms'
    - 'deployment'
#permalink: '/2016/03/25/zentaopms-deployment'
---



 

1. 获取软件

   ```shell
   wget http://sourceforge.net/projects/zentao/files/8.1.3/ZenTaoPMS.8.1.3.zip/download
   ```

2. 将软件解压放到web服务器的目录下

    ![img](/images/img_56f4aa5d219ba-20190917143633356.png)

3. 创建数据库，授权用户

   ```sql
   create database zentao;grant all zentao.* to zentao@localhost identified by 'zentap';
   ```

   

   ![img](/images/img_56f4aa6c69a26-20190917143633077.png)

4. Web访问页面开启安装进程

   访问`http://localhost/zentaopms/www/install.php`，同dz等程序安装相同注意配置时使用授权的账户密码和数据库

   

5. 安装完成后需要破解zentaopms方可正常登录

   通过下面的地址下载loader-wizard：http://www.ioncube.com/loader-wizard/loader-wizard.zip 下载之后，将其解压缩，到web服务器的DocumentRoot下。

   

6. 使用浏览器访问loader-wizard.php文件。该文件会检测当前环境，给出提示解决方法依照步骤处理
  ![img](/images/img_56f4aa815cb3c-20190917143633401.png)


7. 重新web服务启动之后，再次访问loader.php，如果安装成功，系统会提示。

   ![img](/images/img_56f4aa8c14ae0-20190917145027287.png)

   看到这个界面，就表示解密软件已经安装成功了。

8. 再次访问zentaopms的首页测试是否可以正常登录
  ![img](/images/img_56f4aa989af5e-20190917143633212.png)

 

