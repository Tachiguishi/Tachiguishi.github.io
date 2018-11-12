---
layout: post
title:  googletest 101
date:   2018-10-16 19:30:00 +0800
categories: tools
tags: googletest
---

`googletest`是基于`xUnit`结构的一个`c++`单元测试框架，是由Google负责开发维护的一个开源项目。
[开源地址](https://github.com/google/googletest)

## 构建

要使用`googletest`需要有其头文件和`libgtest.a`库。
头文件在下载的源码获得，而库文件则需要自己编译  
关于如何构建官方有详细的[文档](build_googletest), 这里简要说明  

假设所获取的源码位于目录`${GTEST_DIR}`

### 直接编译

* 使用`g++`

```shell
g++ -isystem ${GTEST_DIR}/include -I${GTEST_DIR} \
    -pthread -c ${GTEST_DIR}/src/gtest-all.cc
ar -rv libgtest.a gtest-all.o
```

* 使用`CMake`

```shell
mkdir mybuild       # Create a directory to hold the build output.
cd mybuild
cmake ${GTEST_DIR}  # Generate native build scripts.
```

* 验证是否成功

```shell
cd ${GTEST_DIR}/make
make
./sample1_unittest
```

### 使用`CMake`嵌入到已有项目中

假如已经存在一个`CMake`项目想要使用`googletest`，可以将`googletest`的源码复制到项目目录中，然后使用
`add_subdirectory()`将其添加进取。这样可以保证其与原项目使用相同的编译参数编译，但是这在代码维护上会有些问题。  
可以使用`git submodules`来处理，或者使其嵌入到`CMake`来管理`googletest`。
`git submoudles`可以参见`git`文档，这里不做介绍，下面讲述`CMake`的管理方法

* 创建文件`CMakeLists.txt.in`, 写入以下内容

```cmake
cmake_minimum_required(VERSION 2.8.2)

project(googletest-download NONE)

include(ExternalProject)
ExternalProject_Add(googletest
  GIT_REPOSITORY    https://github.com/google/googletest.git
  GIT_TAG           master
  SOURCE_DIR        "${CMAKE_CURRENT_BINARY_DIR}/googletest-src"
  BINARY_DIR        "${CMAKE_CURRENT_BINARY_DIR}/googletest-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
```

* 修改原有项目的`CMakeList.txt`，添加以下内容

```cmake
# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/googletest-src
                 ${CMAKE_CURRENT_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)

# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
  include_directories("${gtest_SOURCE_DIR}/include")
endif()

# Now simply link against gtest or gtest_main as needed. Eg
add_executable(example example.cpp)
target_link_libraries(example gtest_main)
add_test(NAME example_test COMMAND example)
```

[build_googletest]: https://github.com/google/googletest/blob/master/googletest/README.md
