---
layout: post
title:  Chapter 3 The Activity Lifecycle
date:   2017-02-26 10:25:00 +0800
categories:
  - Reading
  - Android Programming by Chris Stewart
---

![activity lifecycle](https://developer.android.google.cn/images/training/basics/basic-lifecycle.png)

`activity`在生命周期之间的切换由`android`系统自动调用相关函数，而不应该手动去调用这些函数，只要重载相关函数即可

## logging the activity lifecycle

### making log messages

`android.util.log`类可以将日志信息发送到系统级的日志中心。`Log`类有多个记录日志的函数，例如：

```java
public static int d(String tag, String msg)
```

`d`代表`debug`，表示日志级别。第一个参数表示日志来源，通常一个常量，其值未相应的类名。第二个参数即未日志级别  
例如在`QuizActivity`中，

```java
public class QuizActivity extends AppCompatActivity{
  private static final String TAG = "QuizActivity";
}
```

然后在`onCreate()`函数中使用`Log.d()`记录一条日志

```java
@Override
protected void onCreate(Bundle savedInstanceState){
  super.onCreate(savedInstanceState);
  Log.d(TAG, "onCreate(Bundle) called");
  setContentView(R.layout.activity_quiz);
  // ...
}
```

使用同样的方式为其它函数添加日志

```java
@Override
public void onStart(){
  super.onStart();
  Log.d(TAG, "onStart() called");
}

@Override
public void onPause(){
  super.onPause();
  Log.d(TAG, "onPause() called");
}

@Override
public void onResume(){
  super.onResume();
  Log.d(TAG, "onResume() called");
}

@Override
public void onStop(){
  super.onStop();
  Log.d(TAG, "onStop() called");
}

@Override
public void onDestroy(){
  super.onDestroy();
  Log.d(TAG, "onDestroy() called");
}
```

你可以在`Logcat`中查看打印的日志内容

## rotation and the activity lifecycle

当旋转手机时，有竖屏变成横屏，我们可以在日志中看到
`onPause(), onStop(), onDestroy(), onCreate(), onStart(), onResume()`
先后被调用。也就是说在旋转后`QuizActivity`先被销毁然后再次创建了一个新的实例

### device configuration and alternative resource

`device configuration`就是设备的规格参数表，保存有设备的各种配置信息，包括屏幕方向，分辨率，语言等。
有些参数时不会改变的，如分辨率，但有些会改变，如屏幕方向。所以当配置参数改变时，`android`会尝试寻找更适合当前配置的资源。
而载入资源／视图等都是在`onCreate(Bundle)`函数中进行的，所以要先销毁再重新创建`activity`

### create a landscape layout

在`Project`视图中，右键`res`选择`New - Android resource directory`在弹出的对话框中  
`Resource type = layout`, `Source set = main`, `qualifiers = Orientation`,点击`>>`  
此时对话框改变，`Screen orientation = Landscape`，确认此时`Directory name == layout-land`  
上面这些操作就是为了创建`res/layout-land/`文件夹

`-land`后缀时一个[`configuration qualifier`](https://developer.android.google.cn/guide/topics/resources/providing-resources.html)

将`res/layout/activity_quiz.xml`复制到文件夹`res/layout-land`中，并做修改  
使用`FrameLayout`代替`LinearLayout`然后为其它组建添加`android:layout_gravity`属性进行排版

## save data across rotation

可以通过重载下面的函数在旋转时缓存数据

```java
protected void onSaveInstanceState(Bundle outState)
```

该方法通常会在`onPause(), onStop(), onDestroy()`之前被调用，数据被缓存在`Bundle`结构中，
然后在`onCreate(Bundle)`中被读取。  
`Bundle`是存储字符串键与特定类型值映射关系(键值对)的一种结构，
`Bundle`中只能存储基本数据类型和实现了`Seriablizable`接口的对象。

首先添加字符串键

```java
public class QuizActivity extends AppCompatActivity{
  private static final String KEY_INDEX = "index";
}
```

重载`onSaveInstanceState()`函数将`mCurrentIndex`的值保存到相应的键中

```java
@Override
protected void onSaveInstanceState(Bundle savedInstanceState){
  super.onSaveInstanceState(savedInstanceState);
  Log.i(TAG, "onSaveInstanceState");
  savedInstanceState.putInt(KEY_INDEX, mCurrentIndex);
}
```

在`onCreate(Bundle)`中通过键获取缓存的值

```java
@Override
protected void onCreate(Bundle savedInstanceState){
  super.onCreate(savedInstanceState);

  if(savedInstanceState != null){
    mCurrentIndex = savedInstanceState.getInt(KEY_INDEX, 0);
  }
}
```

运行结果来看`lifecycle`函数调用顺序：`onPause(), onSaveInstanceState(), onStop(), onDestroy()`.

按`返回`键会调用`onDestroy()`，而按`Home`键只会调用`onStop()`

## log level

| Log Level | Method | Notes |
| :-------- | :----- | :---- |
| ERROR | Log.e() | Errors |
| WARNING | Log.w() | Warnings |
| INFO | Log.i() | Informational messages |
| EDBUG | Log.d() | Debug output; may be filtered out |
| VERBOSE | Log.v() | For development only |
