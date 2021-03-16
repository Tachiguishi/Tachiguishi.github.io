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

```shell
sudo mount -o uid=pi,gid=pi,iocharset=utf8 /dev/sdb1 /home/pi/lagann
```

`df -T`查看磁盘格式

### 挂载U盘后变为`Read-Only`

1. 使用`dmesg`诊断，查看日志

```
FAT-fs (sda1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
FAT-fs (sda1): error, fat_free_clusters: deleting FAT entry beyond EOF
FAT-fs (sda1): Filesystem has been set read-only
```

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

### Tracker

```
http://tracker.trackerfix.com:80/announce,
udp://9.rarbg.me:2880,
udp://9.rarbg.to:2830,
http://176.113.68.67:6961/announce,
http://176.113.71.60:6961/announce,
http://184.105.151.164:6969/announce,
http://185.83.214.123:6969/announce,
http://51.15.55.204:1337/announce,
http://51.38.230.101:80/announce,
http://93.158.213.92:1337/announce,
http://95.107.48.115:80/announce,
http://aaa.army:8866/announce,
http://bobbialbano.com:6969/announce,
http://derpyradio.net:6969/announce,
http://h4.trakx.nibba.trade:80/announce,
http://open.acgnxtracker.com/announce,
http://opentracker.i2p.rocks:6969/announce,
http://publictracker.ch:6969/announce,
http://retracker.sevstar.net:2710/announce,
http://rt.tace.ru:80/announce,
http://share.camoe.cn:8080/announce,
http://t.acg.rip:6699/announce,
http://t.nyaatracker.com/announce,
http://t.nyaatracker.com:80/announce,
http://t.overflow.biz:6969/announce,
http://tr.cili001.com:8070/announce,
udp://vibe.community:6969/announce,
http://tracker.dler.org:6969/announce,
http://tracker.files.fm:6969/announce,
udp://zephir.monocul.us:6969/announce,
udp://www.midea123.z-media.com.cn:6969/announce,
udp://valakas.rollo.dnsabr.com:2710/announce,
http://tracker.kamigami.org:2710/announce,
http://tracker.lelux.fi:80/announce,
http://tracker.noobsubs.net:80/announce,
http://tracker.skyts.cn:6969/announce,
http://tracker.skyts.net:6969/announce,
http://tracker.ygsub.com:6969/announce,
http://tracker.zerobytes.xyz:1337/announce,
http://tracker1.itzmx.com:8080/announce,
http://tracker2.dler.org:80/announce,
http://tracker2.itzmx.com:6961/announce,
http://tracker3.itzmx.com:6961/announce,
http://tracker4.itzmx.com:2710/announce,
http://vpn.flying-datacenter.de:6969/announce,
http://vps02.net.orel.ru/announce,
http://vps02.net.orel.ru:80/announce,
https://1337.abcvg.info/announce,
https://1337.abcvg.info:443/announce,
https://aaa.army:8866/announce,
https://publictracker.pp.ua:443/announce,
https://tr.ready4.icu:443/announce,
https://tracker.gbitt.info/announce,
https://tracker.hama3.net:443/announce,
https://tracker.imgoingto.icu:443/announce,
https://tracker.lelux.fi:443/announce,
https://tracker.nitrix.me:443/announce,
https://tracker.tamersunion.org:443/announce,
https://trakx.herokuapp.com:443/announce,
https://w.wwwww.wtf:443/announce,
udp://176.113.68.67:6961/announce,
udp://176.113.71.60:6961/announce,
udp://185.181.60.67:80/announce,
udp://185.83.214.123:6969/announce,
udp://207.241.226.111:6969/announce,
udp://207.241.231.226:6969/announce,
udp://208.83.20.20:6969/announce,
udp://212.1.226.176:2710/announce,
udp://212.47.227.58:6969/announce,
udp://217.76.183.53:80/announce,
udp://37.235.174.46:2710/announce,
udp://46.148.18.250:2710/announce,
udp://46.148.18.254:2710/announce,
udp://47.ip-51-68-199.eu:6969/announce,
udp://5.226.148.20:6969/announce,
udp://61626c.net:6969/announce,
udp://62.210.97.59:1337/announce,
udp://6ahddutb1ucc3cp.ru:6969/announce,
udp://91.149.192.31:6969/announce,
udp://91.216.110.52:451/announce,
udp://93.158.213.92:1337/announce,
udp://adm.category5.tv:6969/announce,
udp://admin.videoenpoche.info:6969/announce,
udp://adminion.n-blade.ru:6969/announce,
udp://api.bitumconference.ru:6969/announce,
udp://aruacfilmes.com.br:6969/announce,
udp://bclearning.top:6969/announce,
udp://benouworldtrip.fr:6969/announce,
udp://bioquantum.co.za:6969/announce,
udp://bitsparadise.info:6969/announce,
udp://blokas.io:6969/announce,
udp://bms-hosxp.com:6969/announce,
udp://bubu.mapfactor.com:6969/announce,
udp://cdn-1.gamecoast.org:6969/announce,
udp://cdn-2.gamecoast.org:6969/announce,
udp://chanchan.uchuu.co.uk:6969/announce,
udp://cutiegirl.ru:6969/announce,
udp://daveking.com:6969/announce,
udp://discord.heihachi.pw:6969/announce,
udp://dpiui.reedlan.com:6969/announce,
udp://edu.uifr.ru:6969/announce,
udp://eliastre100.fr:6969/announce,
udp://engplus.ru:6969/announce,
udp://fe.dealclub.de:6969/announce,
udp://forever-tracker.zooki.xyz:6969/announce,
udp://inferno.demonoid.is:3391/announce,
udp://ipv4.tracker.harry.lu:80/announce,
udp://josueunhuit.com:6969/announce,
udp://kanal-4.de:6969/announce,
udp://koli.services:6969/announce,
udp://mail.realliferpg.de:6969/announce,
udp://movies.zsw.ca:6969/announce,
udp://mts.tvbit.co:6969/announce,
udp://nagios.tks.sumy.ua:80/announce,
udp://ns-1.x-fins.com:6969/announce,
udp://open.stealth.si:80/announce,
udp://opentor.org:2710/announce,
udp://opentracker.arg.bz:6969/announce,
udp://opentracker.i2p.rocks:6969/announce,
udp://publictracker.xyz:6969/announce,
udp://qg.lorzl.gq:2710/announce,
udp://retracker.akado-ural.ru:80/announce,
udp://retracker.lanta-net.ru:2710/announce,
udp://retracker.local.msn-net.ru:6969/announce,
udp://retracker.netbynet.ru:2710/announce,
udp://retracker.sevstar.net:2710/announce,
udp://rutorrent.frontline-mod.com:6969/announce,
udp://sd-161673.dedibox.fr:6969/announce,
udp://storage.groupees.com:6969/announce,
udp://teamspeak.value-wolf.org:6969/announce,
udp://torrent.tdjs.tech:6969/announce,
udp://tr.cili001.com:8070/announce,
udp://tracker.0x.tf:6969/announce,
udp://tracker.altrosky.nl:6969/announce,
udp://tracker.blacksparrowmedia.net:6969/announce,
udp://tracker.coppersurfer.tk:6969/announce,
udp://tracker.dler.org:6969/announce,
udp://tracker.ds.is:6969/announce,
udp://tracker.dyne.org:6969/announce,
udp://tracker.filemail.com:6969/announce,
udp://tracker.fortu.io:6969/announce,
udp://tracker.kamigami.org:2710/announce,
udp://tracker.leechers-paradise.org:6969/announce,
udp://tracker.lelux.fi:6969/announce,
udp://tracker.opentrackr.org:1337/announce,
udp://tracker.shkinev.me:6969/announce,
udp://tracker.skynetcloud.site:6969/announce,
udp://tracker.skyts.net:6969/announce,
udp://tracker.tiny-vps.com:6969/announce,
udp://tracker.uw0.xyz:6969/announce,
udp://tracker.v6speed.org:6969/announce,
udp://tracker.vulnix.sh:6969/announce,
udp://tracker.zemoj.com:6969/announce,
udp://tracker.zerobytes.xyz:1337/announce,
udp://tracker.zum.bi:6969/announce,
udp://tracker0.ufibox.com:6969/announce,
udp://tracker2.dler.org:80/announce,
udp://tracker2.itzmx.com:6961/announce,
udp://tracker3.itzmx.com:6961/announce,
udp://tracker6.dler.org:2710/announce,
udp://tracker-udp.gbitt.info:80/announce,
udp://ultra.zt.ua:6969/announce,
udp://us-tracker.publictracker.xyz:6969/announce,
http://share.camoe.cn:8080/announce,
udp://tracker.torrent.eu.org:451/announce,
http://t.nyaatracker.com:80/announce,
udp://tracker.doko.moe:6969/announce,
http://asnet.pw:2710/announce,
udp://thetracker.org:80/announce,
http://tracker.tfile.co:80/announce,
http://pt.lax.mx:80/announce,
udp://santost12.xyz:6969/announce,
https://tracker.bt-hash.com:443/announce,
udp://bt.xxx-tracker.com:2710/announce,
udp://tracker.vanitycore.co:6969/announce,
udp://zephir.monocul.us:6969/announce,
http://grifon.info:80/announce,
http://retracker.spark-rostov.ru:80/announce,
http://tr.kxmp.cf:80/announce,
http://tracker.city9x.com:2710/announce,
udp://bt.aoeex.com:8000/announce,
http://tracker.tfile.me:80/announce,
udp://tracker.tiny-vps.com:6969/announce,
http://retracker.telecom.by:80/announce,
http://tracker.electro-torrent.pl:80/announce,
udp://tracker.tvunderground.org.ru:3218/announce,
udp://tracker.halfchub.club:6969/announce,
udp://retracker.nts.su:2710/announce,
udp://wambo.club:1337/announce,
udp://tracker.dutchtracking.com:6969/announce,
udp://tc.animereactor.ru:8082/announce,
udp://tracker.justseed.it:1337/announce,
udp://tracker.leechers-paradise.org:6969/announce,
udp://tracker.opentrackr.org:1337/announce,
https://open.kickasstracker.com:443/announce,
udp://tracker.coppersurfer.tk:6969/announce,
udp://open.stealth.si:80/announce,
http://retracker.mgts.by:80/announce,
http://retracker.bashtel.ru:80/announce,
udp://inferno.demonoid.pw:3418/announce,
udp://tracker.cypherpunks.ru:6969/announce,
http://tracker.calculate.ru:6969/announce,
udp://tracker.sktorrent.net:6969/announce,
udp://tracker.grepler.com:6969/announce,
udp://tracker.flashtorrents.org:6969/announce,
udp://tracker.yoshi210.com:6969/announce,
udp://tracker.tiny-vps.com:6969/announce,
udp://tracker.internetwarriors.net:1337/announce,
udp://mgtracker.org:2710/announce,
http://tracker.yoshi210.com:6969/announce,
http://tracker.tiny-vps.com:6969/announce,
udp://tracker.filetracker.pl:8089/announce,
udp://tracker.ex.ua:80/announce,
http://mgtracker.org:2710/announce,
udp://tracker.aletorrenty.pl:2710/announce,
http://tracker.filetracker.pl:8089/announce,
http://tracker.ex.ua/announce,
http://mgtracker.org:6969/announce,
http://retracker.krs-ix.ru:80/announce,
udp://tracker2.indowebster.com:6969/announce,
http://thetracker.org:80/announce,
http://tracker.bittor.pw:1337/announce,
udp://tracker.kicks-ass.net:80/announce,
udp://tracker.aletorrenty.pl:2710/announce,
http://tracker.aletorrenty.pl:2710/announce,
http://tracker.bittorrent.am/announce,
udp://tracker.kicks-ass.net:80/announce,
http://tracker.kicks-ass.net/announce,
http://tracker.baravik.org:6970/announce,
http://tracker.dutchtracking.com/announce,
http://tracker.dutchtracking.com:80/announce,
udp://tracker4.piratux.com:6969/announce,
http://tracker.internetwarriors.net:1337/announce,
udp://tracker.skyts.net:6969/announce,
http://tracker.dutchtracking.nl/announce,
http://tracker2.itzmx.com:6961/announce,
http://tracker2.wasabii.com.tw:6969/announce,
udp://tracker.sktorrent.net:6969/announce,
http://www.wareztorrent.com:80/announce,
udp://bt.xxx-tracker.com:2710/announce,
udp://tracker.eddie4.nl:6969/announce,
udp://tracker.grepler.com:6969/announce,
udp://tracker.mg64.net:2710/announce,
udp://tracker.coppersurfer.tk:6969/announce,
http://tracker.opentrackr.org:1337/announce,
http://tracker.dutchtracking.nl:80/announce,
http://tracker.edoardocolombo.eu:6969/announce,
http://tracker.ex.ua:80/announce,
http://tracker.kicks-ass.net:80/announce,
http://tracker.mg64.net:6881/announce,
udp://tracker.flashtorrents.org:6969/announce,
http://tracker.tfile.me/announce,
http://tracker1.wasabii.com.tw:6969/announce,
udp://tracker.bittor.pw:1337/announce,
http://tracker.tvunderground.org.ru:3218/announce,
http://tracker.grepler.com:6969/announce,
udp://tracker.bittor.pw:1337/announce,
http://tracker.flashtorrents.org:6969/announce,
http://retracker.gorcomnet.ru/announce,
udp://tracker.sktorrent.net:6969/announce,
udp://tracker.sktorrent.net:6969,
udp://public.popcorn-tracker.org:6969/announce,
udp://tracker.ilibr.org:80/announce,
udp://tracker.kuroy.me:5944/announce,
udp://tracker.mg64.net:6969/announce,
udp://tracker.cyberia.is:6969/announce,
http://tracker.devil-torrents.pl:80/announce,
udp://tracker2.christianbro.pw:6969/announce,
udp://retracker.lanta-net.ru:2710/announce,
udp://tracker.internetwarriors.net:1337/announce,
udp://ulfbrueggemann.no-ip.org:6969/announce,
http://torrentsmd.eu:8080/announce,
udp://peerfect.org:6969/announce,
udp://tracker.swateam.org.uk:2710/announce,
http://ns349743.ip-91-121-106.eu:80/announce,
http://torrentsmd.me:8080/announce,
http://agusiq-torrents.pl:6969/announce,
http://fxtt.ru:80/announce,
udp://tracker.vanitycore.co:6969/announce,
udp://explodie.org:6969
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
