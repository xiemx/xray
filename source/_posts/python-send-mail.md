---
layout: post
title: 邮件发送脚本
published: true
author: xiemx
comments: true
date: 2016-04-28 03:04:48
tags: [ python, mail ]
categories:
    - python
---
python调用外部smtp服务器发送邮件 
```python
#!/usr/bin/env python

from email.MIMEText import MIMEText
from email.MIMEMultipart import MIMEMultipart
from email.MIMEBase import MIMEBase
from email import Utils,Encoders
import mimetypes,sys
import smtplib

def send_mail(user,passwd,host,context,sub,to_user):
    self_user = user
    self_pass = passwd
    self_smtp = host
    self_sub  = sub
    self_context = context
    self_touser = to_user

    msg = MIMEMultipart()
    msg["To"] = self_touser
    msg["From"] = "xxxx@unxn.com"
    msg["Subject"] = self_sub
    msg["Data"] = Utils.formatdate(localtime = 1)
    
    body = MIMEText(self_context,_subtype = "plain")
    
    msg.attach(body)
    send = smtplib.SMTP(self_smtp)
    send.login(self_user,self_pass)
    send.sendmail(self_user,self_touser,msg.as_string())
    send.close()

if __name__=='__main__':
    host= "smtp.qq.com"
    user= "xxxxxxxx@qq.com"
    passwd= "xxxxxxxx"
    to_user = "xxxxxxxxx@163.com"
    sub = "Log anaylse result!"

    f = open("log","r")
    context = f.read()
    f.close()
    
    send_mail(user,passwd,host,context,sub,to_user)
```