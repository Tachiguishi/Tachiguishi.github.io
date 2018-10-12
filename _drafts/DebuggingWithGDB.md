---
layout: post
title:  Debugging with GDB
date:   2018-10-10 19:30:00 +0800
categories: tools
tags: gdb
---

`GDB(GNU Debugger)`是`Linux`上通用的命令行调试工具，`DDD(Data Display Debugger)`则是`GDB`的`GUI`版本。  
可以使用他们在`Linux`上实现`c/c++`的调试工作

```shell
gdb --version
#=> GNU gdb (GDB) Fedora 8.1.1-3.fc28
ddd --version
#=> GNU DDD 3.3.12 (x86_64-redhat-linux-gnu)
```

## GDB基本命令

* `break line`  
    设置断点
* `clear line`  
    消除断点
* `tbreak line`  
    设置临时断点，即该断点在被出发过一次后就会自动失效
* `run`  
    启动程序
* `next` 与 `step`  
    单步运行一条语句，`step`会进入函数内部，`next`不会
* `continue`  
    继续运行程序直至下一断点或程序结束
* `until`
* `finish`
* `print variablename`
* `watch [variablename | condition]`  
    监视`variablename`值,但其值变化时会自动出发断点。也可以使用条件语句，如`wactch (var > 30)`
* stack frame  
    相关命令: `frame no`, `up`, `down`, `backtrace`
* `-tui`  
    进入`TUI`模式(可以显示源码)，在普通模式下也可以通过按`ctrl+x,ctrl+a`进入`TUI`模式
