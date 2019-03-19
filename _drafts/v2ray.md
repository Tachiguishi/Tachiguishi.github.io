---
layout: post
title:  V2Ray
date:   2018-11-06 19:15:00 +0800
categories: tools
tags: v2ray
---

V2Ray is a unified platform for anti-censorship.  
之所以使用`v2ray`是因为我的`shadowsocks`一个月内连续被封了3次端口。
于是我一方便同时为`shadowsocks`开了多个端口，随机使用，另一方便开始使用`v2ray`.

## 安装

`V2Ray`服务端与客户端使用相同的程序，所以安装方式相同，关键是配置文件的区别

安装需要`su`权限

```shell
bash <(curl -L -s https://install.direct/go.sh)
# or
wget https://install.direct/go.sh && chmod +x go.sh && sudo ./go.sh
```

## 配置

配置文件`/etc/v2ray/config.json`

### 服务端

```json
{
	"log":{
		"loglevel": "warning",
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log"
	},
	"inbounds": [{
		"port": 80,
		"protocol": "vmess",
		"settings": {
			"clients": [{
				"id": "de408bc1-f9d3-526d-1462-76dd94bf5010",
				"alterId": 10
			}]
		}
	}],
	"outbounds": [{
		"protocol": "freedom",
		"settings": {}
	}]
}
```

服务端与客户端的`id`必须一致，类似于密码。  
格式必须是`uuid`的格式，可以使用 `v2ctl uuid`命令生成或者使用网站www.uuidgenerator.net生成

### 客户端

```json
{
	"log":{
		"loglevel": "warning",
		"access": "/var/log/v2rayaccess.log",
		"error": "/var/log/v2rayerror.log"
	},
	"inbounds": [{
		"port": 1080,
		"protocol": "socks",
		"sniffing":{
			"enable": true,
			"destOverride": ["http", "tls"]
		},
		"setting":{
		"auth": "noauth",
		"udp": true
		}
	}],
	"outbounds": [{
		"protocol": "vmess",
		"settings": {
			"vnext":[{
			"address": "server_address",
				"port": 80,
				"users":[{
					"id": "de408bc1-f9d3-526d-1462-76dd94bf5010",
					"alterId": 10
				}]
			}]
		}
	},{
		"protocol": "freedom",
		"settings": {},
		"tag": "direct"
	}],
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [{
			"type": "field",
			"outboundTag": "direct",
			"domain": ["geosite:cn"]
		},{
			"type": "field",
			"ip": ["geoip:private", "geoip:cn"],
			"outboundTag": "direct"
		}]
	}
}
```

`outbounds`中的`tag`为标签名，可以随意取名，供`routing`中的`rules`使用  
`routing`中的`rules`中的`outboundTag`即指定`outbounds`中的`tag`，指明满足哪些条件的流量使用指定`protocol`  
上述`routing`配置中指定局域网与大陆IP使用`direct`所指向的`protocol`,即`freedom`,表示直连

## 运行

在运行前可以需要先检测配置文件是否符合规范

```shell
/usr/bin/v2ray/v2ray -test -config /etc/v2ray/config.json
```

如果配置文件有错误会指出具体那里有错，如果没有问题会出现如下信息：

```
V2Ray 4.18.0 (Po) 20190228
A unified platform for anti-censorship.
Configuration OK.
```

确认配置文件没有问题就可以启动了

```shell
sudo systemctl start v2ray
```

## Mac

```shell
# install
brew tap v2ray/v2ray
brew install v2ray-core

# update
brew update
brew upgrade v2ray-core

# uninstall
brew uninstall v2ray-core
brew untap v2ray/v2ray

# edit the default config
vim /usr/local/etc/v2ray/config.json
# run v2ray-core without starting at login
brew services run v2ray-core
# run v2ray-core and register it to launch at login
brew services start v2ray-core
```

## Android

`Android`平台可以使用`v2rayNG`或`BifrostV`进行配置

注意，`v2ray`启动是会在手机上自动添加一个`VPN`配置，  
如果你手机配置过一个`VPN`，且将这个`VPN`设置为了`Always-on VPN`的话，则会导致无法配置新的`VPN`，  
`v2ray`启动时报错`permission denied to create a VPN service`  
这时只要将那个`VPN`的`Always-on VPN`选项关闭即可

## 广告屏蔽

广告屏蔽是通过客户端的路由配置完成的，所以只需修改客户端的配置即可

首先为客户端的`outbounds`添加一项`blackhole`协议

```json
{
	"protocol": "blackhole",
	"settings": {},
	"tag": "adblock"
}
```

在路由配置项`routing`中添加广告服务的域名，将其导向`blackhole`

```json
{
	"type": "field",
	"outboundTag": "adblock",
	"domain": ["geosite:category-ads"]
}
```

其中`geosite:category-ads`的广告域名列表保存在`/usr/bin/v2ray/geosite.dat`文件中  
改列表由[domain-list-community](https://github.com/v2ray/domain-list-community)负责维护

## Issues

### 不定时断流

```
clntIP:Port1 rejected  v2ray.com/core/proxy/vmess/encoding: failed to read request header > read tcp servIP:Port2->clntIP:Port1: i/o timeout
```

用`Android`手机通过`v2rayNG`在`YouTube`上连播听歌时突然连接断开，然后发现所有外网都连不上。
`v2rayNG`的`tap to check connection`有时能成功，有时不能。  
在到服务端发现`v2rayaccess.log`日志出现上述错误  
同时用身旁处于同一网络环境，相同`V2Ray`客户端配置的电脑能够正常访问外网。  
于是判断问题位于手机上，仅仅重启服务无效，但重启整个`v2rayNG`应用后恢复正常

## Reference

[offical_manual](https://www.v2ray.com)
[简易教程](https://toutyrater.github.io)
