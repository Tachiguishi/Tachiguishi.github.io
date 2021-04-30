---
layout: post
title:  Qt
date:   2019-08-27 19:15:00 +0800
categories: tools
tags: qt
---

## install

```shell
dnf install qt5-qtbase-devel
dnf install qt5-designer
```

`qmake-qt5`

## build with cmake

```cmake
cmake_minimum_required(VERSION 3.1.0)

project(helloworld VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# handle moc automatically for Qt targets.
set(CMAKE_AUTOMOC ON)
# handle rcc automatically for Qt targets
set(CMAKE_AUTORCC ON)
# handle uic automatically for Qt targets
set(CMAKE_AUTOUIC ON)

if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

find_package(Qt5 COMPONENTS Widgets REQUIRED)

add_executable(helloworld
    mainwindow.ui
    mainwindow.cpp
    main.cpp
    resources.qrc
)

target_link_libraries(helloworld Qt5::Widgets)
```

## QDebug

`qDebug()`无输出但`qInfo()`可以正常输出，需修改配置文件

```ini
# ~/.config/QtProject/qtlogging.ini
[Rules]
*.debug=true  # Enable all debug messages
qt.*.debug=false  # Disable qt debug messages, leaving only ours
```
