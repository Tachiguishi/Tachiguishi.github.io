## `Windows`

访问[SQLite下载页面](http://www.sqlite.org/download.html)  
下载`sqlite-dll-win32-*.zip`和`sqlite-tools-win32-*.zip`

### 安装

解压`sqlite-tools-win32-*.zip`，得到其中的`sqlite3.exe`文件  
保存`sqlite3.exe`到一个你喜欢的文件夹，如：`C:\sqlite`  
然后将`C:\sqlite`添加到`PATH`环境变量中

命令行中  
```Bash
$ sqlite3 --version
3.15.1 2016-11-04 12:08:49 1136863c76576110e710dd5d69ab6bf347c65e36
```

> 之前`sqlite3.exe`文件时被放在`sqlite-shell-win32-*.zip`中的

### 动态链接库(dll)  

解压`sqlite-dll-win32-*.zip`，得到`sqlite3.dll`和`sqlite3.def`  
这里提供的SQLite DLL是线程安全的，也就是说，它是定义了THREADSAFE预处理符后编译的。
这样就可以在多线程中使用SQLite，也可以在多线程中使用DLL，安全地在多线程中执行同步操作  
该DLL需要与程序`sqlite3.exe`处于同一文件夹下或者在系统的路径中`C:\Windows\System32`，
或者遵循WINDOWS动态链接库加载时寻找DLL的规则

### 问题  

以上安装可是使`SQLite`中场在`cmd`和`powershell`中运行  
但是在`babun`中无法正常工作。`--help`和`--version`可以执行，`sqlite3`一直卡死

使用`g++ sqlite101.cpp -l sqlite3`编译无法关联到相关的库文件

参考  
[Using SQLite3 with Cygwin](http://superuser.com/questions/253059/using-sqlite3-with-cygwin)
> Cygwin uses Windows pipes to emulate ptys,
so native console program see a pipe where they expect to see a console.
Among other issues, that often causes them to enter non-interactive mode.  
> Cygwin currently doesn't work well with interactive native programs

解决  
* 使用`-interactive`可以运行`sqlite3`进行交互
* 通过`Cygwin`的`setup.exe`安装`sqlite`组件，成功使用`g++ sqlite101.cpp -l sqlite3`编译

### 使用VS编译

[Using SQLite in a Native Visual C++ Application](https://dcravey.wordpress.com/2011/03/21/using-sqlite-in-a-visual-c-application/)

下载`sqlite-amalgamation-*.zip`文件，其中包含`sqlite3.h`, `sqlite3.c`等文件
