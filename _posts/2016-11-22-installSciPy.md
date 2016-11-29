---
layout: post
title:  "Install SciPy"
date:   2016-11-22 16:11:47 +0800
categories: python scipy
---

# install SciPy
[官方安装文档](http://www.scipy.org/install.html)

## 使用`pip`安装

### 安装平台

windows上使用的`babun`

### 安装`numpy`过程

* 创建虚拟环境并激活
```bash
» virtualenv SCompute
» source SCompute/bin/activate
(SCompute) »
```

* 安装`numpy`
```bash
(SCompute) » pip intall numpy
```

* 报错
```bash
SystemError: Cannot compile 'Python.h'. Perhaps you need to install python-dev|python-devel.
```

* 使用`pact`安装缺少的`python-devel`
```bash
» pact install python-devel
```

* 再次安装`numpy`
```bash
(SCompute) » pip intall numpy
```

* 报错
```bash
numpy/core/src/multiarray/numpyos.c:18:21: fatal error: xlocale.h: No such file or directory
```

* 原因
[错误解决参考1](https://forum.alpinelinux.org/forum/general-discussion/xlocaleh)
[错误解决参考2](https://wired-world.com/?p=100)

* 使用`ln -s`[软链接(symbolic link)](http://www.ahlinux.com/start/cmd/582.html)
```bash
(SCompute) » ln -s /usr/include/locale.h /usr/include/xlocale.h
(SCompute) » pip install numpy
» install success
```
> 前提：你已经安装gcc

### 安装`scipy`过程

* 安装`scipy`和`matplotlib`
```bash
(SCompute) » pip install scipy matplotlib
```

* 报错
```
BUILDING MATPLOTLIB
                matplotlib: yes [1.5.3]
                    python: yes [2.7.12 (default, Oct 10 2016, 12:50:22)  [GCC
                            5.4.0]]
                  platform: yes [cygwin]

    REQUIRED DEPENDENCIES AND EXTENSIONS
                     numpy: yes [version 1.11.2]
                  dateutil: yes [dateutil was not found. It is required for date
                            axis support. pip/easy_install may attempt to
                            install it after matplotlib.]
                      pytz: yes [pytz was not found. pip will attempt to install
                            it after matplotlib.]
                    cycler: yes [cycler was not found. pip will attempt to
                            install it after matplotlib.]
                   tornado: yes [tornado was not found. It is required for the
                            WebAgg backend. pip/easy_install may attempt to
                            install it after matplotlib.]
                 pyparsing: yes [pyparsing was not found. It is required for
                            mathtext support. pip/easy_install may attempt to
                            install it after matplotlib.]
                    libagg: yes [pkg-config information for 'libagg' could not
                            be found. Using local copy.]
                  freetype: no  [The C/C++ header for freetype2 (ft2build.h)
                            could not be found.  You may need to install the
                            development package.]
                       png: no  [pkg-config information for 'libpng' could not
                            be found.]
                     qhull: yes [pkg-config information for 'qhull' could not be
                            found. Using local copy.]

    OPTIONAL SUBPACKAGES
               sample_data: yes [installing]
                  toolkits: yes [installing]
                     tests: yes [nose 0.11.1 or later is required to run the
                            matplotlib test suite. Please install it with pip or
                            your preferred tool to run the test suite / mock is
                            required to run the matplotlib test suite. Please
                            install it with pip or your preferred tool to run
                            the test suite]
            toolkits_tests: yes [nose 0.11.1 or later is required to run the
                            matplotlib test suite. Please install it with pip or
                            your preferred tool to run the test suite / mock is
                            required to run the matplotlib test suite. Please
                            install it with pip or your preferred tool to run
                            the test suite]

    OPTIONAL BACKEND EXTENSIONS
                    macosx: no  [Mac OS-X only]
                    qt5agg: no  [PyQt5 not found]
                    qt4agg: no  [PySide not found; PyQt4 not found]
                   gtk3agg: no  [Requires pygobject to be installed.]
                 gtk3cairo: no  [Requires cairocffi or pycairo to be installed.]
                    gtkagg: no  [Requires pygtk]
                     tkagg: yes [installing; run-time loading from Python Tcl /
                            Tk]
                     wxagg: no  [requires wxPython]
                       gtk: no  [Requires pygtk]
                       agg: yes [installing]
                     cairo: no  [cairocffi or pycairo not found]
                 windowing: no  [Microsoft Windows only]

    OPTIONAL LATEX DEPENDENCIES
                    dvipng: yes [version 1.15]
               ghostscript: no
                     latex: yes [version MiKTeX 2.9]
                   pdftops: yes [version 0.46.0]

    OPTIONAL PACKAGE DATA
                      dlls: no  [skipping due to configuration]

    ============================================================================
                            * The following required packages can not be built:
                            * freetype, png
```

* 安装缺少的`freetype`组件
```bash
(SCompute) » pact install libfreetype-devel
```
> 安装`freetype`组件时会同时安装`png`组件

* 再次单独安装`scipy`
```bash
(SCompute) » pip install scipy
```

* 报错
```bash
numpy.distutils.system_info.NotFoundError: no lapack/blas resources found
```

* 安装缺少的`lapack`和`blas`组件
```bash
(SCompute) » pact install libopenblas liblapack-devel
```

* 再次单独安装`scipy`
```bash
(SCompute) » pip install scipy
```

* 报错
```bash
error: library dfftpack has Fortran sources but no Fortran compiler found
```

* 安装缺少的`fortran`组件
```bash
(SCompute) » pact install gcc-fortran
```

* 再次单独安装`scipy`
```bash
(SCompute) » pip install scipy
» install success
```

### 安装其余计算库

```bash
(SCompute) » pip install matplotlib
» install success
(SCompute) » pip install ipython
» install success
(SCompute) » pip install pandas
» install success
(SCompute) » pip install sympy
» install success
(SCompute) » pip install nose
» install success
```

### 遗留问题

`jupyter` 安装出错
```bash
bundled/zeromq/src/signaler.cpp:62:25: fatal error: sys/eventfd.h: No such file or directory
    compilation terminated.
    error: command 'gcc' failed with exit status 1
```
未解决

### 总结

使用`scipy`安装时有很多依赖不会被直接安装，需要手动去安装
只要看清楚安装失败时报错说的是什么组件，然后去安装就行了
错误信息中提到的组件名和安装时使用的组件名很可能不同，比如：
报错中提到缺少`freetype`组件，这时只需要首先使用`find`查找
```bash
» pact find freetype
Searching for installable packages matching freetype:
freetype2-debuginfo
freetype2-demos
libfreetype-devel
libfreetype-doc
libfreetype6
mingw64-i686-freetype2
mingw64-x86_64-freetype2
```
然后选择`libfreetype-devel`安装就可以了
```bash
» pact install libfreetype-devel
```

## windows 安装

### 官方建议1

使用第三方发布的[科学计算库](http://www.scipy.org/install.html#scientific-python-distributions)
如：
* Anaconda: A free distribution for the SciPy stack. Supports Linux, Windows and Mac.
* Enthought Canopy: The free and commercial versions include the core SciPy stack packages. Supports Linux, Windows and Mac.
* Python(x,y): A free distribution including the SciPy stack, based around the Spyder IDE. Windows only.
* WinPython: A free distribution including the SciPy stack. Windows only.
* Pyzo: A free distribution based on Anaconda and the IEP interactive development environment. Supports Linux, Windows and Mac.

### 官方建议2

使用[`packages`](http://www.scipy.org/install.html#windows-packages)
在[该网站](http://www.lfd.uci.edu/~gohlke/pythonlibs/)下载相关的安装包到本地，
然后使用`pip`安装(需指明安装包路径)
```bash
» pip install numpy-1.11.2+mkl-cp27-cp27m-win32.whl
```
> 此方法只可在windows中安装的独立python有效，不支持`cygwin`
