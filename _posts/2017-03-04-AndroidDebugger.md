---
layout: post
title:  Chapter 4 Debugging Android Apps
date:   2017-03-04 16:57:00 +0800
categories:
  - Reading
  - Android
---

## Exceptions and stack traces

`exception`错误信息会自动打印到`logcat`中显示。
而对于其它bug则可以使用日志或者设置断定进行调试，要启动调试不能点`运行`而应该点击`调试`来启动程序  
在`Run - View Breakpoints`中可以将`exception`设置为断点。

## Android specific Debugging

### Using Android Lint

`Android Lint`是静态分析器，可以检测出`java`编译器无法察觉的错误。  
可以手动启动`Analyze - Inspect Code`，然后在弹出的对话框中选择要检视的范围

### Issues with the R class

如果在`build`过程中出错，可以考虑如下解决方案

* 检测资源文件中`xml`文件的格式是否正确，是否有错误的字段
* 清理你的项目`Build - Clean Project`
* 同步`gradle`文件 `Tools - Android - Sync Project with Gradle Files`
* 运行`Android Lint`
