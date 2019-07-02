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

按`Ctrl + R`进入`RAID`配置界面，创建一个新的虚拟磁盘即可

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
`QT_QPA_FONTDIR=fonts_directory`

## 获取文件大小 

通过将文件当前位置指示符放在文件尾部，然后再获取指示符的位置来得到文件大小(bytes)

```c++
#include <ifstream>

int main(){
    std::ifstream file("somthing.dat");
    file.seekg(0, std::ios::end);
    int fileSize = file.tellg();
    return 0;
}
```

## 判断文件是否存在

```c++
#ifdef _WIN32
#include <io.h>
int _access(const char *path, int mode);
#else
#include <unistd.h>
int access(const char *path, int mode);
#endif

```

其中`mode`有如下集中取值：

| mode | checks file for |
| ---- | --------------- |
|  0   | existence only  |
|  2   | write permission|
|  4   | read premission |
|  6   | rean and write permission |

```c++
void main(){
    if((_access("somefile.txt", 0)) == -1){
        printf("can't file file.");
    }
    else if((_access("somefile.txt", 4)) == -1){
        printf("have no read permission");
    }
}

```

## 获取当前路径

```c++
#ifdef _WIN32
#include <direct.h>
#else
#include <unistd.h>
#endif
char* getcwd(char* buffer, int len);
```

## zh-hans.tld-list.com

botbouncer

```
Hello.  Your IP address is making unauthorized automated requests to this website and access has been temporarily banned.

To restore immediate access to this website, make the following Bitcoin payment within the next 2 hours:

Bitcoin Address: 1FY5WqsMkpaUfdj3XiRSi36dQPX8QcFnyo
Bitcoin Amount: 0.0005 BTC
QR code: https://chart.googleapis.com/chart?cht=qr&chs=300x300&chld=L|2&chl=bitcoin%3A1FY5WqsMkpaUfdj3XiRSi36dQPX8QcFnyo%3Famount%3D0.0005

After your full payment has reached 0 confirmation(s), your IP address will be granted access for a month.

If you think this ban was made in error, or you are experiencing problems with your payment, please contact the website administrator at example@example.com and include your IP address in your message. 

```


## Oracle数据库连接缓慢

在`ubuntu 16.04`上直接使用`sqlplus user/pwd`连接数据库很快，而使用`sqlplus sys/sys@local`连接十分缓慢，可能需要5~10分钟才能连上。  
数据库位于本机，确认`listener.ora`和`tnsnames.ora`配置无问题。`tnsnames.ora`中指定的`host`地址为`localhost`,
`/etc/hosts`中有配置语句`127.0.0.1 localhost`。  
结果发现是`/etc/resolv.conf`中有语句`nameserver 127.0.1.1`, 将该条语句删除后数据库连接速度恢复正常，瞬间连上。

`/etc/resolv.conf`中的配置是有`NetworkManager`写入的，修改其配置文件`/etc/NetworkManager/NetworkManager.conf`，删除`dns=dnsmasq`。
这样就可以保证以后即使重启电脑后`/etc/resolv.conf`文件中也不会出新`nameserver 127.0.1.1`

## Ubuntu 16.04 将启动器移到屏幕底部

```shell
gsettings set com.canonical.Unity.Launcher launcher-position Bottom 
```

## vs code markdown preview 不显示svg图片

这是与有`vs code`的安全机制引起的，运行`Markdown: Change preview security settings`命令，将安全级别改为`Allow insecure content`即可  
参考文档: [MarkDown preview secutiry](https://code.visualstudio.com/docs/languages/markdown#_markdown-preview-security)

## `WIN32, _WIN32, _WIN64`

`WIN32`定义在头文件`minwindef.h`中，只要使用了头文件`Windows.h`则程序中就会定义

```c++
#ifndef WIN32
    #define WIN32
#endif
```

正因为时这种方式定义的，所以可以使用`#undefine WIN32`来取消定义

而`_WIN32`与`_WIN64`是微软的`Visual C++ compiler`预定义的宏，无法修改。微软[官方文档](https://msdn.microsoft.com/en-us/library/b0084kay.aspx)解释为：

> _WIN32 Defined as 1 when the compilation target is 32-bit ARM, 64-bit ARM, x86, or x64. Otherwise, undefined.
> _WIN64 Defined as 1 when the compilation target is 64-bit ARM or x64. Otherwise, undefined.

所以一般情况下使用`Visual Studio`编译时`_WIN32`始终被定义，而`_WIN64`只在编译64位程序时才会被定义

## userdel -r username 删除用户

## vs code doesn't read .bash_profile in integrated terminal

vs code 集成终端运行时报错`bash: __git_ps1: command not found...`,而我正常的终端却没有问题

这是因为`VS Code`启动的是一个`no-login`的`shell`，所以只会载入`.bashrc`。  
所以可以将`source ～/git-prompt.sh`等相关配置放入`.bashrc`中  
或者在`VS Code`中修改配置`"terminal.integrated.shellArgs.linux": ["-l"]`，使继承终端变成一个`login`终端，
这样就会载入`.bash_profile`文件

[官方文档](https://code.visualstudio.com/docs/editor/integrated-terminal#_shell-arguments)

## 库连接顺序

`gcc`默认开启`--as-needed`，这导致会自动忽略没有被使用的库，即使你添加到了`-llibname`参数中。  
所以当连接库的参数顺序不对时会导致有些库被认为不需要而被忽略  
静态库连接错误: `undefined reference to: xxx`  
动态库连接错误: `undefined symbol: xxx`  

### 解决方法

* 在项目开发过层中尽量让lib是垂直关系，避免循环依赖；越是底层的库，越是往后面写
* 通过-(和-)强制repeat，让一些库重复查找保证其被编译，但是这样会浪费一些时间

[参考](https://www.cnblogs.com/OCaml/archive/2012/06/18/2554086.html)

## sqlplus创建表错误

```
unknown command beginning "CONSTRAINT..." - rest of line ignored.
```

这是由于`constraint`语句与前面的语句间包含空行，导致语句解析错误。只要将空行删除即可  
[参考](https://stackoverflow.com/questions/27112678/oracle-unknown-command-constraint)

## socket connect 非阻塞

## socket recv 返回值

## ubuntu 631端口被监听

## 主机与vm虚拟机中的ubuntu互相拷贝

```
apt-get install open-vm-tools open-vm-tools-desktop
```

## shell终端光标消失

假如Linux下光标消失，不要急：

echo -e "\033[?25l"  隐藏光标
echo -e "\033[?25h" 显示光标

## `linux`挂载`windows`共享文件夹

假设`windows`共享文件夹地址`//win-hostname/wingman`(可以通过在`windows`上参看文件夹共享属性得知),用户名`win-user`,密码`win-passwd`。  
`linux`上用来挂载的文件夹`/run/media/linux-user/wingman`(该文件夹需要是已经存在的)
则可以使用如下命令来进行挂载

```shell
sudo mount -t cifs -o username=win-user,password=win-passwd //win-hostname/wingman /run/media/linux-user/wingman
```

这样挂载的文件夹中所有文件的属性都是`-rwxr-xr-x`,也就是说是无法修改文件，也无法向文件家中添加文件。而且也无法修改权限。  
使用`id linux-user`获取用户的`gid`与`uid`，加入都为`1000`。然后修改上述命令为

```shell
sudo mount -t cifs -o username=win-user,password=win-passwd,gid=1000,uid=1000 //win-hostname/wingman /run/media/linux-user/wingman
```

卸载

```shell
umount /run/media/linux-user/wingman
```

[reference](https://blog.csdn.net/tojohnonly/article/details/71374984)

## 查看以安装软件

1、rpm包安装的，可以用rpm -qa看到，如果要查找某软件包是否安装，用 rpm -qa | grep “软件或者包的名字”。

[root@hexuweb102 ~] rpm -qa | grep ruby

2、以deb包安装的，可以用dpkg -l能看到。如果是查找指定软件包，用dpkg -l | grep “软件或者包的名字”；

[root@hexuweb102~]dpkg-l|grepruby

3、yum方法安装的，可以用yum list installed查找，如果是查找指定包，命令后加 | grep “软件名或者包名”；

[root@hexuweb102 ~] yum list installed | grep ruby

4、如果是以源码包自己编译安装的，例如.tar.gz或者tar.bz2形式的，这个只能看可执行文件是否存在了，

上面两种方法都看不到这种源码形式安装的包。如果是以root用户安装的，可执行程序通常都在/sbin:/usr/bin目录下


## 查看开机启动

```shell
# 列出所有开机启动项
systemctl list-unit-files
# 检测某个服务(如redis.service)是否开机启动
systemctl is-enabled redis.service
```

## sqlplus 中文乱码

```sql
select userenv('language') from dual;
```

## vs code encoding

```json
{
    // 自动检测文件编码
    "files.autoGuessEncoding": true,
    // 文件默认打开编码
    "files.encoding": "utf8",
    // 针对语言选择编码
    "[markdown]": {
        "files.encoding": "utf8"
    }
}
```

## pipenv

安装

```
pip install --user pipenv
```

运行`python -m site --user-base`获取`pipenv`执行文件路径,并添加其到`~/.bashrc`文件中

```shell
python -m site --user-base
#=> /Users/jetbrains/.local
# 添加环境便令
echo "export PATH=$PATH:Users/jetbrains/.local/bin" >> ~/.bashrc
```

[Configuring Pipenv Environment](https://www.jetbrains.com/help/pycharm/pipenv.html)

## powerline

### fonts

```shell
sudo apt-get install fonts-powerline
dnf install powerline-fonts
```

vscode 的终端无法正确显示：  
下载[Menlo for Powerline](https://github.com/abertsch/Menlo-for-Powerline)字体  
放入`~/.fonts`文件夹，运行`fc-cache -vf ~/.fonts`  

修改`vscode`配置

```json
{
"terminal.integrated.fontFamily": "Menlo for Powerline"
}
```

spectrum_ls

## win7 路由表

```shell
# 查看
route print
# 添加(-p表示永久路由)
route add 192.168.2.0 mask 255.255.255.0 192.168.1.1 -p
```

## zsh: no match found

在使用`gtest`的`--gtest_filter=testgroup.*`测试用例筛选功能时, `zsh`报错`no match found:`  
这是由于`zsh`自己尝试解析你的命令参数中的`*`引起的。所以只要不让`zsh`解析而是直接将参数传给`gtest`就可以了

在`~/.zshrc`添加如下配置：

```
setopt no_nomatch
```

[参考](https://www.jianshu.com/p/87d85593006e)

但是即使这样在`vscode`中调试时还是不行，这是只要在`launch.json`的参数配置的传递值用`''`包含即可

```json
{
    "args": ["'--gtest_filter=testgroup.*'"]
}
```

## MySQL 8.0版本连接报错：Could not create connection to database server

MySQL8.0版本需要更换驱动为“com.mysql.cj.jdbc.Driver”，之前的“com.mysql.jdbc.Driver”已经不能在MySQL 8.0版本使用了

[参考](https://www.cnblogs.com/smiler/p/9963773.html)


## ubuntu界面卡死

find xorg pid and kill it

```shell
ps -t tty7
sudo kill pid
```

## linux wifi manager

```shell
# show saved connections
nmcli c
# list available wifi hotspots
nmcli d wifi list
# connect
nmcli d wifi connect <WiFiSSID> password <WiFiPassword> [iface <WiFiInterface]>
# another connect
nmcli c up id <SavedWiFiConn>
```

[reference](https//askubuntu.com/questions/461825/how-to-connect-to-wifi-from-the-command-line)

## tracker-miner-fs eating up my memory

## centos7 IP

查看

```shell
ip addr
```

虽然插了网线，但是网卡显示并没有被分配`IP`。

在`/etc/sysconfig/network-scripts/`目录下找到网卡名命名文件  
打开发现最后一行`ONBOOT=no`，将其修改为`ONBOOT=yes`  
重启网络`service network restart`  
重新使用`ip addr`发现已经被分配`IP`

## centos 安装图形界面

### 安装

可以先用`yum grouplist`查看可以安装的桌面环境有哪些

```shell
yum groupinstall "X Window System"
# Gnome
yum groupinstall "GNOME Desktop"
# KDE
yum groupinstall "KDE Plasma Workspaces"
```

### 卸载

```shell
yum groupremove "GNOME Desktop"
```

### 切换图形界面

```shell
switchdesk gnome
switchdesk kde
```

### 修改默认图形界面

`/etc/sysconfig/desktop`文件中添加或修改`DESKTOP="GNOME"`或`DESKTOP="KDE"`  
或  
在`~`目录下添加`.xinitrc`，文件内容为`gnome-session`或`startkde`

### 修改启动方式

```shell
# 命令模式
systemctl set-default multi-user.target
# 图形模式
systemctl set-default graphical.target
```

### 查看当前桌面环境

```shell
echo $DESKTOP_SESSION
```

## 通信分析工具

### [WireEdit](https://wireedit.com/)

一款可视化网络数据包编辑工具

### TraceWrangler

抓取的数据包中包含很多网络环境的隐私信息，在需要和别人分享数据包时很不安全。
该款软件可以清楚这些隐私信息

### Tcpreplay

重新发送抓取到的数据包，查看实际反映，利于调试

### NetworkMiner

### CapTipper

### ngrep 数据包的grep

## linux 分辨率

`/etc/X11/xorg.conf`

```conf
Section "Screen"
  Depth 24
  SubSection "Display"
    Depth 24
    Modes "1920x1080"
  EndSubSection
EndSection
```

```shell
xrandr
cvt 1440 900
xrandr --newmode cvt-output
xrandr --addmode monitor-name mode-name
xrandr --output monitor-name --mode mode-name
```

## linux 切换显示器

```shell
#查看显示器列表
xrandr -q
#关闭笔记本显示器，使用VGA-1
xrandr --output VGA-1  --output LVDS-1 --off
#设置VGA-1为主显示器，在笔记本显示器左
xrandr --output VGA-1 --primary --left-of LVDS-1
```

## 关闭ipv6

```shell
sysctl net.ipv6.conf.all.disable_ipv6=1
```

## sed

```shell
a \
text   Append text, which has each embedded newline preceded by a backslash.
i \
text   Insert text, which has each embedded newline preceded by a backslash.
```

## debian 安装缺少固件

在[这里](http://cdimage.debian.org/cdimage/unofficial/non-free/firmware/)下载固件然后放入U盘启动盘根目录的`/firmware`文件夹下即可

[参考](https://www.debian.org/releases/stable/mips/ch06s04.html.zh-cn)

## golang

```shell
dnf install go
```

### vscode 安装 go 插件失败

失败原因是由于网络封锁导致线路不稳定，但是可以通过手动的方式自己下载

```shell
mkdir -p $GOPATH/src/golang.org/x
cd $GOPATH/src/golang.org/x
git clone --depth=1 https://github.com/golang/tools.git
git clone --depth=1 https://github.com/golang/lint.git
# 完成以上步骤后，执行
go get golang.org/x/lint/golint
# 或者打开vscode，让vscode自行安装
```

## pps state mismatchxra

## xfce 显示器配置

`~/.config/xfce4/xfconf/xfce-perchannel-xmls/displays.xml`

```xml
<?xml version="1.0" encoding="UTF-8">

<channel name="displays" version="1.0">
	<property name="Default" type="empty">
		<property name="eDP-1" type="string" value="1. PTN 24&quot;">
			<property name="Active" type="bool" value="false">
		</property>
	</property>
</channel>
```

## grub配置

```conf
GRUB_DEFAULT=0
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
```

1. `GRUB_DEFAULT=0`  
设置默认启动项，在`grub.cfg`文件中按`menuentry`顺序开始计数，从0开始计数。比如要默认从第四个菜单项启动,数字改为3,若改为 saved,则默认为上次启动项。
2. `GRUB_HIDDEN_TIMEOUT=0`  
此配置将影响grub菜单显示。若设置此选项为一个常数，则将在此时间内隐藏菜单而显示引导画面。菜单将会被隐藏，如果注释掉该行，即：(`#GRUB_HIDDEN_TIMEOUT=0`)。则grub菜单能够显示，等待用户的选择，以决定进入那个系统或内核。GRUB2第一次执行时将会寻找其他操作系统。若没有其他操作系统被检测到,菜单将会配置为隐藏。若辨认出其他操作系统,菜单将会显示。若是大于 0 的整数,系统将会依此配置的秒数暂停,但不会显示菜单。0 则菜单不会显示,也不会有延迟。使用者可以在启动时按住 `SHIFT` 键不放以强制显示菜单。启动过程中,系统将会检查 SHIFT 键状态。若无法辨识按键状态,会有一个短时间的延迟让使用者可通过按下 ESC 键来显示菜单。 
3. `GRUB_HIDDEN_TIMEOUT_QUIET=true`  
true不显示倒计时。屏幕将会是空白的。false 在 `GRUB_HIDDEN_TIMEOUT` 中配置的时间,空白屏幕上会有一个倒数计时器。 
4. `GRUB_TIMEOUT=10`  
此命令将顺从 `GRUB_HIDDEN_TIMEOUT` 配置,除非 `GRUB_HIDDEN_TIMEOUT` 被注释掉(#)。若 `GRUB_HIDDEN_TIMEOUT` 启用,则当菜单显示时,`GRUB_TIMEOUT` 将会只执行一次。配置此值为 `-1` 将会导致菜单一直显示,直到用户选择。GRUB2菜单默认为隐藏,除非其他操作系统被系统检测到。若没有其他操作系统,此行将会被注释掉,除非使用者修改它。为了在每次启动时显示菜单,去掉此行的注释并使用 1 或更大的值。
5. `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`  
`quiet`的意思是内核启动时候简化提示信息，`splash`是以图形界面启动

## tcping

检测目标`host`的某个端口是否开启。  
一般情况下会很快返回`open`或`close`结果，如果`timeout`且确认网络无异常则很可能是被墙了

```shell
tcping -t 10 host port
```

## ubuntu 网络设置

`/etc/NetwordManager/system-connections/`  
该文件夹中的配置文件是通过图形界面配置的

`/etc/network/interfaces`  该文件是通过`ifconfig`命令配置的

```shell
ifconfig
ifconfig eth0 192.168.1.0 netmask 255.255.255.0
service network-manager restart
```

## Oracle INS-32031 INS-32033

```shell
chown -R oracle:oinstall/home/oracle
```

## install oracle11g2.32bit error

```log
Checking Temp space: must be greater than 80 MB. Actual 2225807 MB Passed
Checking swap space: must be greater than 150 MB. Actual 4095 MB Passed

The number of files bootstrapped for the jre is 0.

The number of files bootstrapped for the oui is 0.
```

`runInstaller`在检测`pre-require`通过后直接退出，产生上述日志。出错原因是由于硬盘空间>2TB,所以只要将剩余空间减少到2TB以下即可

## oracle11g2.32bit 安装 DBCA 时出错

安装`DBCA`时出错`ORA-00443`, 显示有后台程序停止运行，尝试几次均是如此但每次停止的程序不同。

根据[帖子](https://community.oracle.com/message/11280469)中的描述，很可能是内存(16G)太大引起的。  
于是取下一个内存条(剩余8g), 并在`/etc/sysctl.conf`中设置`kernel.shmmax = 8589934590`  
成功

## curl wget 走代理

```shell
curl -x http://127.0.0.1:1080 github.com
wget -e "http_proxy=127.0.0.1:1080" github.com
```

## 最新整理在线接收短信验证码注册码大全

1、ReceivingSms
https://www.receivingsms.com

2、blacktel
https://www.blacktel.io

3、spoofBox
http://www.spoofbox.com

4、SmsReceiver
https://sms.ndtan.net

5、OnlineSIM
http://onlinesim.ru

6、Twilio
https://www.twilio.com

7、PdfLibr
https://www.pdflibr.com

8、SELLAITE
http://sms.sellaite.com

9、Receive SMS Online
http://receive-sms-online.com

10、Free Online Phone
https://www.freeonlinephone.org

11、RECEIVE SMS ONLINE
https://www.receivesmsonline.net

12、Receive FREE SMS online
http://receivefreesms.com

13、Receive-SMS
https://receive-sms.com

14、Receive SMS online for Free
https://sms-online.co/receive-free-sms

15、Free SMS Numbers Online
https://smsnumbersonline.com

16、Receive a SMS Online
https://receive-a-sms.com

17、Receive SMS Online for FREE
https://www.receive-sms-online.info

18、SMSReceiveFree
https://smsreceivefree.com

## ssh反向代理

A机器IP: `A.A.A.A` ssh监听端口: `22`, 用户名`alice`, 位于内网  
B及其IP: `B.B.B.B` ssh监听端口: `33`, 用户名`finn`, 位于公网  

在A上建立与B端口`3000`的反向代理

```shell
ssh -fCNR 3000:localhost:22 finn@B.B.B.B -p33
```

在B上做端口转发，将连接到`44`的ssh连接转向`3000`，而`3000`与A已经建立连接，从而最终转向A

```shell
ssh -fCNL *:44:localhost:3000 localhost -p33
```

在可以访问B但无法访问A的机器C上使用(登录到A)：

```shell
ssh alice@B.B.B.B -p44
```

## qmake多个项目编译

c++编译时回先生成`*.o`文件，文件名与原文件相同。当如果不同文件夹的文件名刚好相同，
在编译第二个文件时检测到已经有`*.o`文件(第一个文件生成的)了，所以不会再次生成，
致使第二个文件不会编译

## Pure Virtual Function Called

产生这个问题的原因主要有二：
1) 在基类的构造函数里调用了纯虚函数，这个很容易理解，很显然俺们的项目里不会有这种低级错误，否则一跑就crash… 总之，这个错误就忽略了；
2) 某个继承类的对象调用一个虚函数时，这个对象已经被析构了，这时可能是内存错误，也可能是pure virtual function called

第一个不太可能会犯，而第二个可能由于变量定义顺序的关系，导致要使用的变量提前析构如：

```c++
class A{
public:

    virtual ~A(){};

    virtual void fun() = 0;
};

class B : public A{
public:
    virtual ~B(){}

    virtual void fun() override{
        // do something
    }
};

class C{
public:
    ～C(){
        m_pa->fun();
    }

    void SetA(A* p){
        m_pa = p;
    }

private:
    A* m_pa;
};


int main(){
    C c;
    B b;
    c.SetA(&b);
}
```

上述程序中，在`c`析构前`b`已经析构，导致`c`析构中调用`b`的函数出错

## Ubuntu kidle_inject进程占用CPU过高

临时解决方案  

```shell
sudo rmmod intel_powerclamp
```

即可解决，但是重启后失效

永久解决方案

```shell
echo "blacklist intel_powerclamp" | sudo tee /etc/modprobe.d/disable-powerclamp.conf
```

[参考](https://blog.csdn.net/soslinken/article/details/54892752)

## Ubuntu ifconfig only show lo

已确认网卡硬件完好且正确插入，故排除硬件故障。

使用`sudo lshw -C network`查看硬件信息，显示:

```shell
sudo lshw -C network
# *-network UNCLAIMED
#    description: Ethernet controller
#    product: Ethernet Connection (5) I219-LM
#    vendor: Intel Corporation
```

由`UNCLAIMED`可知系统没有合适驱动，故只需安装合适的驱动即可  

根据网卡型号`Intel 219-LM`找到其驱动[e1000e](https://downloadcenter.intel.com/download/15817/Intel-Network-Adapter-Driver-for-PCIe-Intel-Gigabit-Ethernet-Network-Connections-Under-Linux-)

安装驱动后重启即可

```shell
tar xfv e1000e.tar.gz
cd e1000e/src
sudo make install
sudo reboot
```

## Ubuntu 基本tool

git, vim, oh-my-zsh, cmake, vscode, tmux


## Ubuntu telegeram

```shell
sudo add-apt-repository ppa:atareao/telegram
sudo apt-get update
sudo apt-get install telegram
```
