---
layout: post
title:  VPS Settings
date:   2018-11-17 19:30:00 +0800
categories: tools
tags: googletest
---

此文用于记录我在所购买的`VPS`进行的各种操作

## Wyvern.Avalon

操作系统: `Centos 7 x86_64 bbr`

### 添加新用户

```shell
useradd username
passwd username
visudo #=> add: username ALL=(ALL) ALL
```

### ssh

#### 添加端口

`/etc/ssh/sshd_config`
```shell
Port $Ganmu 
```

重启`sshd`: `service sshd restart`  
检查新添加的端口是否被监听: `ss -lnt`

#### 添加公钥

```shell
# run in local pc
ssh -p $Ganmu user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
# or
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host -p $Ganmu
```

添加`ssh`链接配置

`~/.ssh/config`

```conf
Host Wyvern_Avalon
    HostName hostname||hostip
    Port $Ganmu
    User root
    PreferredAuthentications publickey
    IdentityFile /Users/Spike/.ssh/id_rsa
```

### denyhosts

```shell
yum update
yum install epel-release
yum install denyhosts
```

### basic tools

```shell
yum install vim
yum install tmux
```
