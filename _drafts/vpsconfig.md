---
layout: post
title:  VPS Settings
date:   2018-11-17 19:30:00 +0800
categories: trivia
tags: vps
---

此文用于记录我在所购买的`VPS`进行的各种操作

## Wyvern.Avalon

操作系统: `ubuntu-22.04-x86_64`
```shell
uname -a
# Linux wyvern.avalon 5.15.0-25-generic #25-Ubuntu SMP Wed Mar 30 15:54:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
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
PasswordAuthentication no ; 禁止密码登录
PermitRootLogin yes  ; 允许root用户登录
Port $Ganmu 
```

重启`sshd`: `systemctl restart sshd`  
检查新添加的端口是否被监听: `ss -lnt`

#### 添加公钥

```shell
# generate ssh key pair
ssh-keygen -t ecdsa -C "label for this key pair"
# or
ssh-keygen -t ed25519 -C "label for this key pair"
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
    IdentityFile ~/.ssh/id_rsa
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
apt install vim zsh tmux git

# install oh-my-zsh
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
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
# check for current timezone
ls -l /etc/localtime
# query available timezone
timedatectl list-timezones
# change timezone
sudo timedatectl set-timezone America/Los_Angeles
```

```shell
sudo touch /etc/profile.d/timezone.sh
sudo echo "export TZ='Asia/Shanghai'" > /etc/profile.d/timezone.sh
```
