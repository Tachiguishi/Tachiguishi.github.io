---
layout: post
title:  VPS Settings
date:   2018-11-17 19:30:00 +0800
categories: trivia
tags: vps
---

此文用于记录我在所购买的`VPS`进行的各种操作

## Wyvern.Avalon

操作系统: `Centos 7 x86_64 bbr`
```shell
cat /etc/redhat-release
# CentOS Linux release 7.5.1804 (Core)
uname -r
# 4.19.2-1.el7.elrepo.x86_64
```

### 添加新用户

```shell
useradd username
passwd username
visudo #=> add: username ALL=(ALL) ALL
```

### ssh

#### 修改`sshd`配置

`/etc/ssh/sshd_config`
```conf
PermitRootLogin no
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
    User username
    PreferredAuthentications publickey
    IdentityFile /Users/Spike/.ssh/id_rsa
```

为了防止`Bad owner or permissions on .ssh/config`错误，
`~/.ssh/config`文件的权限必须是`600`,即该文件的读写权限只能该用户一人独有

### denyhosts

```shell
yum update
yum install python-ipaddr
wget https://github.com/denyhosts/denyhosts/archive/v3.1.tar.gz
sudo python setup.py install
cd /etc/init.d
ln -s /usr/bin/daemon-control-list denyhosts
service denyhosts start
```

### basic tools

```shell
yum install vim
yum install tmux
yum install wget
yum install git
```

### others

#### PS1

```bashrc
green="\[\033[0;32m\]"
blue="\[\033[0;34m\]"
purple="\[\033[0;35m\]"
reset="\[\033[0m\]"
export PS1="[$purple\u$reset@$green\h $blue\W$reset]$ "
```

#### Timezone

```shell
sudo touch /etc/profile.d/timezone.sh
sudo echo "export TZ='Asia/Shanghai'" > /etc/profile.d/timezone.sh
```
