---
layout: post
title:  CMake 101
date:   2018-11-14 19:30:00 +0800
categories: tools
tags: cmake
---

`CMake`即`Cross platform Make`,是一个开源跨平台的自动化构建工具。
它并不直接编辑生成最终的运行程序或库，而是产生用于构件最终成品的其它工具的构建文件，然后再用其它工具进行真正的构建。
以此实现跨平台的结果。  
在`Linux`上通常结合`make`一起使用。

> `make`教程[跟我一起写Makefile](https://seisman.github.io/how-to-write-makefile/index.html)

## 安装

```shell
dnf install cmake
```

## 示例

```cmake
cmake_minimum_required (VERSION 3.0)

project(demoProject)

add_executable(demo main.cpp)

target_link_libraries(demo pthread)
```

```
cmake . && make
```

## 常用配置

### compiler

```shell
cmake -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
```

或

```cmake
set(CMAKE_C_COMPILER "clang")
set(CMAKE_CXX_COMPILER "clang++" )
```

### flags

```cmake
set(CMAKE_CXX_FLAGS "-Wall")
```

### debug/release

```shell
mkdir Release
cd Release
cmake -DCMAKE_BUILD_TYPE=Release ..
make

mkdir Debug
cd Debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

### library path

```cmake
include_directories("D:/OSGEARTH/include")
link_directories("D:/OSGEARTH/lib")
link_libraries(osg osgDB osgViewer)
```

### output path

```cmake
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
```

### use environment variables

```
$ENV{variable}
```

```cmake
add_custom_command(
  OUTPUT testData.cpp
  COMMAND reswrap 
  ARGS    testData.src > testData.cpp
  DEPENDS testData.src 
)
set_property(SOURCE unit-tests.cpp APPEND PROPERTY OBJECT_DEPENDS testData.cpp)

add_executable(app main.cpp)
add_executable(tests unit-tests.cpp)
```

## 常用变量

* `CMAKE_CURRENT_LIST_DIR`: `CMakeLists.txt`文件所在目录
* `CMAKE_CURRENT_BINARY_DIR`: `cmake`当前工作目录
* `CMAKE_SOURCE_DIR`: 源文件根目录(运行cmake时指定的目录)

## reference

* [入门教程](http://www.hahack.com/codes/cmake/)
