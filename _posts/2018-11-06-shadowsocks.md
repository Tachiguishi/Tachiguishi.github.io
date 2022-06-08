---
layout: post
title:  Shadowsocks
date:   2018-11-06 19:15:00 +0800
categories: tools
tags: shadowsocks
---

`Shadowsocks`是一种基于`Socks5`代理方式的加密传输协议，被广泛用于突破`GFW`。  
`Shadowsocks`分为服务器端和客户端，在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理

## Client

以`Fedora 28`上安装`shadowsocks-libev`为例([官方文档][shadowsocks-libev-fedora-copr])：

### 安装

```shell
dnf copr enable -y libredhat/shadowsocks
dnf install -y shadowsocks-libev simple-obfs
```

[shadowsocks-libev-fedora-copr]: https://copr.fedorainfracloud.org/coprs/outman/shadowsocks-libev/

使用python版本

```shell
# 直接使用此种方式安装的sslocal为2.*版本,不支持aes-256-gcm加密方式
sudo dnf isntall python3-shadowsocks
# 使用pip直接安装3.*版本
pip install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
```

### 配置

`/etc/shadowsocks-libev/config.json`

```json
{
	"server":"remote-shadowsocks-server-ip-addr",
	"server_port":443,
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"your-passwd",
	"timeout":300,
	"method":"chacha20",
	"fast_open":false,
	"workers":1
}
```

### 启动

* 命令行

```shell
ss-local -c /etc/shadowsocks-libev/config.json
```

* 守护进程

```shell
# 其中的`config`为配置文件的文件名
sudo systemctl start shadowsocks-libev-local@config.service
```

* 开机自启动

```shell
sudo systemctl enable shadowsocks-libev-local@config.service
```

* 启动失败：

> error while loading shared libraries: libmbedcrypto.so.2:
> cannot open shared object file: No such file or directory

```shell
ldd /bin/ss-local
#=>linux-vdso.so.1 (0x00007ffec466e000)
#=>libm.so.6 => /lib64/libm.so.6 (0x00007fd49e392000)
#=>libev.so.4 => /lib64/libev.so.4 (0x00007fd49e183000)
#=>libcares.so.2 => /lib64/libcares.so.2 (0x00007fd49df71000)
#=>libsodium.so.23 => /lib64/libsodium.so.23 (0x00007fd49dd1e000)
#=>libmbedcrypto.so.2 => not found
#=>libpcre.so.1 => /lib64/libpcre.so.1 (0x00007fd49d84e000)
#=>libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fd49d62f000)
#=>libc.so.6 => /lib64/libc.so.6 (0x00007fd49d270000)
#=>/lib64/ld-linux-x86-64.so.2 (0x00007fd49e957000)
#=>libpkcs11-helper.so.1 => /lib64/libpkcs11-helper.so.1 (0x00007fd49d053000)
#=>libdl.so.2 => /lib64/libdl.so.2 (0x00007fd49ce4f000)
#=>libcrypto.so.1.1 => /lib64/libcrypto.so.1.1 (0x00007fd49c9be000)
#=>libz.so.1 => /lib64/libz.so.1 (0x00007fd49c7a7000)

ll /lib64/libmbedcrypto.so.*
#=>-rwxr-xr-x. 1 root root 402976 Sep 19 17:43 /lib64/libmbedcrypto.so.2.13.0
#=>lrwxrwxrwx. 1 root root     23 Sep 19 17:43 /lib64/libmbedcrypto.so.3 -> libmbedcrypto.so.2.13.0

# 发现存在libmbedcrypto.so库，只是版本不同，所以建立软连接使其成功找到库文件
ln -s /lib64/libmbedcrypto.so.3 /lib64/libmbedcrypto.so.2
```

### 使用

客户端启动后，会开始监听配置文件中的`local_address`和`local_port`，默认是`127.0.0.1:1080`。  
需要走代理的程序只要配置其`socks host`代理为该`local_address:local_port`即可，
或者直接配置系统代理

![ss_network_proxy](https://imgur.com/9bDnZTI.png)

### PAC

使用全局代理不利于访问国内网站，这时可以使用`PAC(Proxy auto-config)`模式。  

1. 使用`genpac`生成`pac`文件

	```shell
	pip install genpac
	genpac --proxy="SOCKS5 127.0.0.1:1080" \
		--gfwlist-proxy="SOCKS5 127.0.0.1:1080" \
		-o autoproxy.pac \
		--gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
	```

2. 配置  
	代理模式使用`Automatic`, `Configuration URL`填写生成`pac`文件的路径, 如`file:///etc/shadowsocks-libev/autoproxy.pac`

## 参考文档

* [Shadowsocks_Arch_wiki](https://wiki.archlinux.org/index.php/Shadowsocks_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
* [科学上网漫游指南](https://lvii.gitbooks.io/outman/content/)

## OpenWrt

### 安装

```shell
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
```

其中`shadowsocks-libev`和`PC`版没区别，`luci-app-shadowsocks`用于在路由器的网页版配置界面控制`shadowsocks-libev`  
在路由器上配置`shadowsocks`主要是将其配置成`透明代理`(`ss-redir`)，这样连接此路由的设备无需作任何配置机可通过`shadowsocks`上网

`cat /proc/cpuinfo` 查看路由器型号，根据型号在[OpenWrt官网](https://openwrt.org)查询`Package Arch`  
我所使用的路由器是[Youku YK1](https://openwrt.org/toh/hwdata/youku/youku_yk1),对应的型号是`mipsel_24kc`

[reference](http://www.cashqian.net/blog/001472734000655b3d2e0db753848d39a052bc75220291f000)

[透明代理](https://www.zfl9.com/ss-redir.html)

## Server

服务端推荐使用秋水逸冰的[四合一安装脚本](https://teddysun.com/486.html)  
服务端可以同时启用多个端口，每个端口拥有独立的密码，方便多人使用，[参考](https://teddysun.com/532.html)  

```json
{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "9000":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
```
