# 2018-09-11

## 制作U盘启动盘

使用`UltraISO.v.9.6.2.3059`制作`ubuntu-16.04.5-desktop-amd64`U盘启动盘失败，报错:  

```
Failed to load ldlinux.c32
Boot failed: please change disks and press a key to continue.
```

网上传言是`UltraISO`的问题，使用[Universal-USB-Installer-1.9.6.6](http://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/)  
或  
[unetbootin](http://unetbootin.github.io/)


但我使用上述两种刻录软件也都不成功


尝试使用[win32diskimager](http://sourceforge.net/projects/win32diskimager/files/latest/download)  
成功

## Ubuntu

### 产看版本

* cat /etc/issue  ====>>> Ubuntu 8.04 /n /l
* sudo lsb_release -a 
Distributor ID:    Ubuntu
Description:    Ubuntu 8.04
Release:    8.04
Codename:    hardy

## 编译qt3.3.2 in ubuntu16.04

[下载链接](http://download.qt.io/archive/qt/3/qt-x11-free-3.3.2.tar.gz)

查看配置选项

```shell
./confiugre -help
```

根据显示的说明选择配置参数

```shell
./configure -thread
sudo make
```

### x11/xlib.h: no such file or directory

apt-get install libx11-dev

### ../include/qvaluelist.h, ../include/qmap.h: 'ptrdiff_t' does not name a type

add `#include <stddef.h>`

### /usr/bin/ld: cannot find -lXext

apt-get install libxext-dev (require x11proto-xext-dev)

## 华为服务器安装Ubuntu

The creation of swap space in partition #5 of SCSI1 (2,0,0) (sda) failed.

产生问题的原因是由于无法读取服务器上的硬盘。但是在开机启动时系统显示是可以正确检测到硬盘的，而且我换了几块硬盘后依然无效，所以排除是硬盘的问题。而这个U盘启动盘在其他PC机上使用时完全没有问题的，所以也判处安装盘的问题。接下来就只有华为服务器的问题了，于是我上网了华为服务器的[文档](https://forum.huawei.com/enterprise/zh/thread-333163-1-1.html)，发现里面提到了`RAID`这个概念。由于我的服务器原本有三块硬盘，一些原因去除了两块硬盘，且是LSI SAS3108控制器，所以需要重新配置`RAID`。于是根据[华为文档](http://support.huawei.com/enterprise/zh/doc/EDOC1000004345?section=j07s)中的说明，重新配置后成功安装

# 2018-09-13

## ubuntu freezing frequently

```shell
sudo vi /etc/default/grub
#There is a line in that: GRUB_CMDLINE_LINUX_DEFAULT="quiet splash" (like this), replace with: GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_idle.max_cstate=1"
#save and quit
sudo update-grub
sudo reboot
```

## ubuntu ssh server

`ubuntu`自带`ssh-client`端，但没有`server`端，要自行安装

```
sudo apt-get install openssh-server
```

## linux 显卡

```shell
lspci -vnn | grep VGA -A 12 
# 或者
lshw -C display
```

查看`modinfo i915` `glxinfo`

`Linux 4.15.0-29-generic #31~16.04.1-Ubuntu SMP Wed Jul 18 08:54:04 UTC 2018 x86_64 x86_64 x86_64 GUN/Linux`  
`Linux 4.15.0-34-generic #37~16.04.1-Ubuntu SMP Tue Aug 28 10:44:06 UTC 2018 x86_64 x86_64 x86_64 GUN/Linux`


插入独立显卡后需要在`BIOS`界面切换所使用的显卡

## 使用独立显卡后`BIOS`检测不到硬盘，导致无法启动

在`BIOS`中关闭`串口重定向(console serial redirect)`

## login-loop after installing nvidia-384


## fedora shadowsocks

```shell
dnf copr enable -y libredhat/shadowsocks
dnf install -y shadowsocks-libev simple-obfs

sudo systemctl start shadowsocks-libev-local@config.service
sudo systemctl enable shadowsocks-libev-local@config.service
```

## fedora gitkraken

```shell
#!/bin/bash

# Enter /opt folder (common folder for user installed programs)
# This script assumes you have proper permissions on /opt
cd /opt

# Download GitKraken
wget https://release.gitkraken.com/linux/gitkraken-amd64.tar.gz

# Extract the Kraken into /opt directory
tar -xvzf gitkraken-amd64.tar.gz

# Remove tar.gz
rm gitkraken-amd64.tar.gz

# Add gitkraken to PATH
echo "export PATH=\$PATH:/opt/gitkraken" >> ~/.bashrc
source ~/.bashrc

# Download gitkraken launcher icon
wget http://img.informer.com/icons_mac/png/128/422/422255.png
mv 422255.png ./gitkraken/icon.png

# Create desktop entry
sudo touch /usr/share/applications/gitkraken.desktop

# copy the following contents into gitkraken.desktop file:
echo "
[Desktop Entry]
Name=GitKraken
Comment=Git Flow
Exec=/opt/gitkraken/gitkraken
Icon=/opt/gitkraken/icon.png
Terminal=false
Type=Application
Encoding=UTF-8
Categories=Utility;Development;" | sudo tee -a /usr/share/applications/gitkraken.desktop >/dev/null
```

```shell
# I did it this way:
cd /opt/gitkraken
ln -s /usr/lib64/libcurl.so.4 libcurl.so.4
ln -s /usr/lib64/libcurl.so.4 /usr/lib64/libcurl-gnutls.so.4

# Thanks for the script.
```

```shell
#!/bin/bash

# Download GitKraken
wget https://release.gitkraken.com/linux/gitkraken-amd64.tar.gz

# copy the downloaded file into /opt directory
cp gitkraken-amd64.tar.gz /opt/

cd /opt

# Extract the Kraken into /opt directory
tar -xvzf gitkraken-amd64.tar.gz

# you can apply ownership for a specific user too
# chown -R user:group /opt/gitkraken

# Add gitkraken to PATH
echo "export PATH=\$PATH:/opt/gitkraken" >> ~/.bashrc
source ~/.bashrc

# sudo ln -s /usr/lib64/libcurl.so.4 /opt/gitkraken/libcurl-gnutls.so.4
sudo ln -s /usr/lib64/libcurl.so.4 /usr/lib64/libcurl-gnutls.so.4

# Create gitkraken launcher icon
# download icon here: http://img.informer.com/icons_mac/png/128/422/422255.png
# or here: https://drive.google.com/file/d/0B-3KQ_ohu-RFVkJyS1Zfa2NLSVE/view
wget http://img.informer.com/icons_mac/png/128/422/422255.png -o gitkraken-icon.png

mv gitkraken-icon.png /opt/gitkraken/

cd /usr/share/applications

cat > gitkraken.desktop <<EOL
[Desktop Entry]
Name=GitKraken
Comment=Git Flow
Exec=/opt/gitkraken/gitkraken
Icon=/opt/gitkraken/gitkraken-icon.png
Terminal=false
Type=Application
Encoding=UTF-8
Categories=Utility;Development;
EOL

# save it, and voilá!
```

```shell
sudo apt-get install meld;            (install meld)
git config --global merge.tool meld;  (set Git merge tool default as meld)

# 使用
git mergetool
```

## linux debug

逻辑错误用log，内存错误用gdb，单元测试用gtest，编译器用clang，log框架用log4cplus，性能热点用gprof，这样就没有搞不定的bug

补充一条，内存错误用valgrind

Debuggers (gdb), Profilers (gprof, valgrind)


## open terminal in nautilus

```shell
sudo dnf install gnome-terminal-nautilus
nautilus -q
```

## fedora 删除多余内核

```shell
# 查看当前内核
uname -r
 
# 查看所有内核
rpm -qa kernel

# 删除指定内核kernel-4.15.17-200.fc26.x86_64
dnf remove kernel-4.15.17-200.fc26.x86_64
```

重启后启动项多余的选项也会自动被删除

## fedora 修改启动项顺序

```shell
# 产看所有启动项完成名称,menuentry后跟着的就是完整名称
cat /boot/grub2/grub.cfg | grep menuentry

# 修改默认启动项()
grub2-set-default 'Fedora (4.16.3-301.fc28.x86_64) 28 (Workstation Edition)'

# 查看默认启动项(验证修改结果)
grub2-editenv list

# 生成配置文件使修改生效
grub2-mkconfig -o /boot/grub2/grub.cfg
```


## socket error 10038 无效的套接字

使用`send`时出错，说明使用的套接字是无效的

## install Oracle11g-32bit in Ubuntu16.04.05-32bit

安装过程中报错`direct GOT relocation R_386_GOT32 against`,导致`genclntsh`脚本无法生成`libclntsh.so.11.1`库，
网上查阅[资料][binutils_compatibility]说很可能是`binutils`版本兼容问题，  
系统原生自带版本`2.26.1-1ubuntu1~16.04.6`，降级到`2.24-5ubuntu14.2`后依然不行，  
错误信息变为`unrecognized relocation (0x2b) in section '.init'`  
[网上传言][binutils_bug]说这是`binutils`的一个`bug`可以通过升级其版本解决  
于是安装`2.26-8ubuntu2`版本解决，成功生成`libclntsh.so.11.1`

之后报错`rdbms/lib/config.o file not recognized file truncated`导致`irman`与`ioracle`无法生成，
此时只需根据[这篇文章][chang_env_rdbms.mk]里的方法
修改`rdbms/lib/env_rdbms.mk`文件即可。  
> 注意，这篇文章是安装`oracle12`，所以要将`-lnnz12`改为`-lnnz11`

[binutils_compatibility]: https://jhartman.pl/2018/04/24/690/
[binutils_bug]: https://github.com/dirkvdb/ps3netsrv--/issues/13
[chang_env_rdbms.mk]: http://db.geeksinsight.com/2014/10/02/ora-12547-tns-lost-contact-on-dbca-in-12c-with-oel-7-uek/


## 编译qt5.3.1

### 运行`./configure`时`could not find qmake configuration file linux-g++`  

文件是存在的，只是没找到，那么就是路径的问题。通常直接解压官网下载下的源码路径不会有错，没找到是因为路径中有中文

可以将文件夹转移到纯英文目录下，  
或者添加参数指明路径:`./configure -prefix $PWD/qtbase`。这样虽然能配置成功，但之后`make`时依然会因为中文路径出问题，所以建议使用前一种解决方式

### 运行qt5程序时需要添加环境变量

`QT_QPA_PLATFORM_PLUGIN_PATH=qtbase/plugins/platforms`
