---
layout: post
title:  Linux 远程桌面
date:   2019-01-27 19:30:00 +0800
categories: sundry
tags: linux
---

`Linux`远程连接最方便的当然是通过`ssh`,但是其对于桌面系统却不使用。这时就需要用到`VNC`, `RDP`, `SPICE`。  
本文介绍的是在`Fedora`上使用`Boxes`远程连接一台局域网内的`Ubuntu`

## VNC

### 在Ubuntu上

```shell
## 安装
sudo apt-get install vnc4server
## 启动(初次启动时会让你设定连接密码)
vncserver
## 关闭
vncserver -kill :1
## 启动时指定分辨率
vncserver -geometry 1600x900
```

配置文件`~/.vnc/xstartup`

### 在Fedora上

`Boxes`上使用的连接`URL`: `vnc://remotehost:5901`
