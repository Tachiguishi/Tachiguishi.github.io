---
layout: post
title:  Linux Benchmark
date:   2019-01-10 19:30:00 +0800
categories: tools
tags: linux
---

## UnixBench

[UnixBench](https://github.com/kdlucas/byte-unixbench)类unix系（Unix，BSD，Linux）统下的性能测试工具，一个开源工具。
测试包含了系统调用、读写、进程、2D、3D、管道、运算、C库等系统基准性能。
它的优点在于提供了对系统性能的一种评价体系，为系统评分，如此方便对系统作对比测试；但在网络性能测试欠缺。

### 编译

为了测试图形需要将`Makefile`中的`GRAPHIC_TESTS = defined`取消注释，并安装`x11`与`openGL`开发库，然后编译`make`。

```shell
## no file <X11/Xlib.h>
dnf install libXext-devel
apt-get install libxext-dev
## no file <GL/gl.h>
dnf install mesa-libGL-devel
apt-get install libgl1-mesa-dev
```

`## undefined reference to 'sincos'`  
在`Makefile`的`GL_LIBS`向后添加`-lm`

### 运行

```shell
# 执行system测试方法
Run
# 执行graphic测试方法
Run graphics
# 执行system,graphics测试方法
Run gindex
```

`Can't locate Time/HiRes.pm`  

```shell
dnf install perl-devel
```

或

```shell
perl -MCPAN -e shell 
cpan[2]> install Time::HiRes 
cpan[3]> exit
```

`can't exec x11perf`  

```shell
yum install xorg-x11-apps
```

运行`graphics`测试时需要`root`权限

测试结果放在`./results`目录下

## sysbench

### 安装

```shell
# 安装EPEL源
dnf install epel-release
dnf install sysbench
sysbench --version
#=> sysbench 1.0.12
```
