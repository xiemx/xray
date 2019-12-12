---
layout: post
title: vagrant创建多个虚拟机
published: true
author: xiemx
comments: true
date: 2017-02-09 06:02:30
tags: [ ]
categories:
    - vagrant
#permalink: '/2017/02/09/vagrant-mulit-vm'
---
建立多台VM，並且让他们之间能够相通信，一台是应用服务器、一台是DB服务器,参考如下

```markdown
Vagrant.configure("2") do |config|
  config.vm.define :web do |web|
    web.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "web", "--memory", "512"]
    end
    web.vm.box = "base"
    web.vm.hostname = "xiemx-website"
    web.vm.network :private_network, ip: "10.11.1.1"
  end

  config.vm.define :db do |db|
    db.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "db", "--memory", "512"]
    end
    db.vm.box = "base"
    db.vm.hostname = "xiemx-db"
    db.vm.network :private_network, ip: "10.11.1.2"
  end
end
```