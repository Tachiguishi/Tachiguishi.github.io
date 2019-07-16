---
layout: post
title:  Raspberry Pi
date:   2019-04-18 19:15:00 +0800
categories: tools
tags: raspberryPi
---

## 系统安装

[参考文档](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

```shell
sudo dd bs=16M if=2019-04-08-raspbian-stretch-lite.img of=/dev/sdb1
# 107+1 records in
# 107+1 records out
# 1803550720 bytes (1.8 GB, 1.7 GiB) copied, 192.328 s, 9.4 MB/s
```

系统复制进SD卡后插入Raspberry Pi启动，初始账号密码pi/raspberry  
登陆后运行`sudo raspi-config`进行配置

## 静态IP

`/etc/dhcpcd.conf`

```conf
interface eth0
inform 192.168.2.50
static routers=192.168.2.1
static domain_name_servers=192.168.2.1 8.8.8.8
```

## 配置称路由器

https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md

```shell
sudo apt-get install hostapd
```

`raspap-webgui`

## 透明代理

```shell
sudo apt-get install shadowsocks-libev
```

## 使用U盘

插入U盘后在`/dev/`目录下会出现一个`/dev/sd*`类似的文件，也可以使用`sudo fdisk -l`查找。  
假设U盘的设备文件名为`/dev/sda1`  
创建一个文件夹用于挂载：`mkdir ~/guren`  
运行命令载入U盘：`sudo mount -o uid=pi,gid=pi /dev/sda1 /home/pi/guren`  
使用完毕后弹出：`sudo umount /home/pi/guren`

挂载后的U盘乱码：-> 在挂载时使用`-o iocharset=utf8`参数

`/etc/fstab`

```
/dev/sda1 /mnt/chen auto defaults,iocharset=utf8,noexec,umask=0000 0 0
```
