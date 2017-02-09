---
layout: post
title:  Install ruby devkit
date:   2017-02-09 10:30:00 +0800
categories:
  - tools
  - ruby
---

## 准备

* 确定你已经成功安装ruby

```shell
$ ruby -v
ruby 2.3.1p112 (2016-04-26 revision 54768) [x64-mingw32]
```

* 在[这里](http://rubyinstaller.org/downloads)下载符合你情况的development kit  
我电脑是64位系统，所以下载`DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe`

## 安装

双击`DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe`解压到指定目录`D:\DevKit-mingw64`。
你可以根据自己的情况选择自己喜欢的文件夹。  
解压完成后在命令行中运行如下命令

```shell
> cd <DEVKIT_INSTALL_DIR>   # 你的解压路径
> ruby dk.rb init
#生成config.yml，这里会检查将要添加DevKit支持的Ruby列表，只支持通过RubyInstaller安装的Ruby
#如果这里列出的Ruby与你的要求不符，可以手动修改
> ruby dk.rb review  #检查要添加DevKit支持的Ruby列表是否有误，可以略过
> ruby dk.rb install
[INFO] Updating convenience notice gem override for 'C:/Ruby192'
[INFO] Installing 'C:/Ruby192/lib/ruby/site_ruby/devkit.rb'
```

## 参考资料

* [windows下安装devkit](http://rubyer.me/blog/134/)
