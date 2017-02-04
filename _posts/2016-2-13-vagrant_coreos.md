---
layout: post
title: vagrant搭建coreos实验环境
categories: [linux]
tags: coreos
---

<!--break-->

﻿
Vagrant是虚拟机管理工具，可以方便的添加和启动虚拟机，准备实验环境。

官方提供了coreos的vagrant配置。

```
git clone https://github.com/coreos/coreos-vagrant.git
cd coreos-vagrant
```

### 修改配置

进入coreos-vagrant目录，将 user-data.sample 和 config.rb.sample 两个文件各拷贝一份，并去掉 .sample 后缀，得到 user-data 和 config.rb 文件。 修改 config.rb 文件，配置 $num_instances 和 $update_channel 这两个参数。
config.rb文件：

```
# Official CoreOS channel from which updates should be downloaded
$update_channel=’stable’
```

user-data文件：
我们需要获取一个新的token：

```
curl  http://discovery.etcd.io/new
```


会得到一个类似https://discovery.etcd.io/5480377e1e51f25e11dd78f525ba1122的地址。
把这个地址替换user-data文件中的token：

```
discovery:  https://discovery.etcd.io/5480377e1e51f25e11dd78f525ba1122
```

### 下载coreos虚拟机

下载coreos的box文件，并添加到vagrant

```
dlw@dlw:coreos-vagrant$ wget http://storage.core-os.net/coreos/amd64-usr/master/coreos_production_vagrant.box
dlw@dlw:coreos-vagrant$ vagrant box add coreos-stable coreos_production_vagrant.box
```

### 初始化虚拟机

```
dlw@dlw:coreos-vagrant$ vagrant up --provision
Bringing machine 'core-01' up with 'virtualbox' provider...
==> core-01: Importing base box 'coreos-stable'...
==> core-01: Matching MAC address for NAT networking...
==> core-01: Setting the name of the VM: coreos-vagrant_core-01_1455326263773_77079
==> core-01: Clearing any previously set network interfaces...
==> core-01: Preparing network interfaces based on configuration...
    core-01: Adapter 1: nat
    core-01: Adapter 2: hostonly
==> core-01: Forwarding ports...
    core-01: 22 (guest) => 2222 (host) (adapter 1)
==> core-01: Running 'pre-boot' VM customizations...
==> core-01: Booting VM...
==> core-01: Waiting for machine to boot. This may take a few minutes...
    core-01: SSH address: 127.0.0.1:2222
    core-01: SSH username: core
    core-01: SSH auth method: private key
    core-01: Warning: Connection timeout. Retrying...
    core-01: Warning: Connection timeout. Retrying...
==> core-01: Machine booted and ready!
==> core-01: Setting hostname...
==> core-01: Configuring and enabling network interfaces...
==> core-01: Running provisioner: file...
==> core-01: Running provisioner: shell...
    core-01: Running: inline script
```

### 配置ssh

```
dlw@dlw:coreos-vagrant$ vagrant ssh-config --host core-01
Host core-01
  HostName 127.0.0.1
  User core
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile "/home/dlw/.vagrant.d/insecure_private_key"
  IdentitiesOnly yes
  LogLevel FATAL
```

### ssh登录

```
dlw@dlw:coreos-vagrant$ vagrant ssh core-01
CoreOS developer (955.0.0+2016-02-11-0204)
core@core-01 ~ $ 
```



```
core@core-01 ~ $ fleetctl list-machines
MACHINE		IP		METADATA
276ea6f8...	172.17.8.101	-
```
