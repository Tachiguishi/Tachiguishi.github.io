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

## Aria2

### 安装

```shell
apt install aria2
```

### 配置

`.config/aria2/aria2.config`

```shell
#后台运行
daemon=true
#用户名
#rpc-user=user
#密码
#rpc-passwd=passwd
#设置加密的密钥
rpc-secret=secret
#允许rpc
enable-rpc=true
#允许所有来源, web界面跨域权限需要
rpc-allow-origin-all=true
#是否启用https加密，启用之后要设置公钥,私钥的文件路径
#rpc-secure=true
#启用加密设置公钥
#rpc-certificate=/home/pi/.config/aria2/example.crt
#启用加密设置私钥
#rpc-private-key=/home/pi/.config/aria2/example.key
#允许外部访问，false的话只监听本地端口
rpc-listen-all=true
#RPC端口, 仅当默认端口被占用时修改
#rpc-listen-port=6800
#最大同时下载数(任务数), 路由建议值: 3
max-concurrent-downloads=5
#断点续传
continue=true
#同服务器连接数
max-connection-per-server=5
#最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
min-split-size=10M
#单文件最大线程数, 路由建议值: 5
split=10
#下载速度限制
max-overall-download-limit=4000K
#单文件速度限制
max-download-limit=0
#上传速度限制
max-overall-upload-limit=2000K
#单文件速度限制
max-upload-limit=0
#断开速度过慢的连接
#lowest-speed-limit=0
#验证用，需要1.16.1之后的release版本
#referer=*
#文件保存路径, 默认为当前启动位置(我的是外置设备，请自行坐相应修改)
dir=/home/pi/gurren
#文件缓存, 使用内置的文件缓存, 如果你不相信Linux内核文件缓存和磁盘内置缓存时使用, 需要1.16及以上版本
#disk-cache=0
#另一种Linux文件缓存方式, 使用前确保您使用的内核支持此选项, 需要1.15及以上版本(?)
#enable-mmap=true
#文件预分配, 能有效降低文件碎片, 提高磁盘性能. 缺点是预分配时间较长
#所需时间 none < falloc ? trunc << prealloc, falloc和trunc需要文件系统和内核支持
file-allocation=falloc
#不进行证书校验
check-certificate=false
#保存下载会话
save-session=/home/pi/.config/aria2/aria2.session
input-file=/home/pi/.config/aria2/aria2.session
#断电续传
save-session-interval=60
```

### 开机启动

`/lib/systemd/system/aria.service`

```config
[Unit]
Description=Aria2 Service
After=network.target

[Service]
User=pi
Type=forking
ExecStart=/usr/bin/aria2c --conf-path=/home/pi/.config/aria2/aria2.config

[Install]
WantedBy=multi-user.target
```

```shell
sudo systemctl daemon-reload
sudo systemctl enable aria
```

### web管理界面AriaNg
