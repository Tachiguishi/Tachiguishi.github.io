---
layout: post
title:  "Install g++ on Windows"
date:   2016-11-15 16:11:22 +0800
categories: g++
---

## 使用`cygwin`

* [How to Install the Latest GCC on Windows](http://preshing.com/20141108/how-to-install-the-latest-gcc-on-windows/)
* [Windows中配置g++编译环境最简单方法](http://blog.csdn.net/leonsc/article/details/5853614)
* [Installing c++/g++ on Windows](http://www1.cmc.edu/pages/faculty/alee/g++/g++.html)

## 使用`Mingw`

* [参考1](http://www.drangon.org/mingw/)
* [参考2](http://mingw-w64.sourceforge.net/)

## 使用`Babun`

* 安装
```Bash
pact install gcc-g++
```

* 卸载
```Bash
pact remove gcc-g++
pact remove gcc-core
```

### 安装成功后试运行

创建文件`hello.cpp`
```C++
#include <iostream>
using namespace std;
int main()
{
    cout<<"Hello,World!\n"<<endl;
    cin.get();
    return 0;
}
```

编译
```Bash
g++ hello.cpp -o Hi
```

报错
```Bash
fatal error: stddef.h: No such file or directory compilation terminated.
```

原因
`gcc-core`和`gcc-g++`版本不统一

解决
运行`pact update gcc-core gcc-g++`

参考
[Compiling using g++ #191](https://github.com/babun/babun/issues/191)
[Compilation error: “stddef.h: No such file or directory”](http://stackoverflow.com/questions/31600600/compilation-error-stddef-h-no-such-file-or-directory)

重新运行，成功。获得文件`Hi.exe`

运行`Hi.exe`
```Bash
./Hi.exe
```

得到结果
```Bash
Hello,World!
```

使用`C++11`编译
```Bash
g++ -std=c++11 hello.cpp -o Hi
```

报错
```Bash
In file included from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/ext/string_conversions.h:41:0,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/bits/basic_string.h:5249,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/string:52,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/bits/locale_classes.h:40,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/bits/ios_base.h:41,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/ios:42,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/ostream:38,
                 from /usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/iostream:39,
                 from hello.cpp:1:
/usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/cstdlib:126:11: error: ‘::at_quick_exit’ has not been declared
   using ::at_quick_exit;
           ^
/usr/lib/gcc/i686-pc-cygwin/5.4.0/include/c++/cstdlib:149:11: error: ‘::quick_exit’ has not been declared
   using ::quick_exit;
           ^
```

原因
不明，各种`pact update`无效。之后重新安装`babun`依然是同样的问题
可以参考[这篇帖子](https://github.com/juho-p/fatty/issues/12)

解决

下载`cygwin`安装程序，使用其安装`gcc package`依然无效

最后通过安装`gcc --version 5.3.0`解决问题，估计是`g++`版本的问题

```Bash
{ ~ }  » g++ --version
g++ (GCC) 5.3.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

{ ~ }  » gcc --version
gcc (GCC) 5.3.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

总结
`gcc --version 5.4.0`可能有些问题，使用之前的版本就可以了
可以通过`cygwin`安装程序安装旧版本，`babun`的`pact`似乎无法回溯到旧版本

### 后记

安装`g++`几天后发现`g++`自动升级了

```Bash
{ ~ }  » g++ --version                                                           ~
g++ (GCC) 5.4.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

但是一切编译都正常。我凌乱了


4.9.2
5.4.0
