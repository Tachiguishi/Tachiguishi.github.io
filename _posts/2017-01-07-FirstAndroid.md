---
layout: post
title:  Chapter 1 Your First Android Application
date:   2017-01-07 17:10:00 +0800
categories:
  - Reading
  - Android Programming by Chris Stewart
---

这一章将要写一个叫"GeoQuiz"的程序

## 基本结构

* `activity`  
  用于管理界面上的数据内容，她是`Activity`类的一个实例。你将写一个`Activity`的子类
* `layout`  
  用于定义界面上出现什么元素以及元素的位置。它是一个xml格式的文件

## Laying Out the User Interface

自动生成的`activity_quiz.xml`文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.avalon.ash.geoquiz.QuizActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
</RelativeLayout>
```

可以看到其中包括了两个widget(组件)，`RelativeLayout`和`TextView`  
每个widget都是`View`类或其子类的一个实例

修改`activity_quiz.xml`文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="24dp"
        android:text="@string/question_text"/>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/true_button"/>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/false_button"/>
    </LinearLayout>
</LinearLayout>
```
此时编译器会显示三处错误，这是因为我们还没有定义三个`android:text`所引用的内容

### The view hierarchy

<pre>
|- LinearLayout
  |- TextView
  |- LinearLayout
    |- Button
    |- Button
</pre>

根元素为`LinearLayout`，作为根元素，必须声明其空间`http://schemas.android.com/apk/res/android`  
`LinearLayout`继承自`View`的子类`GroupView`。表明其包含的组件排列为一行或一列  
`GroupView`的其它子类还有`FrameLayout`，`TableLayout`，`RelativeLayout`

### Widget attributes

#### android:layout_width and android:layout_height

`android:layout_width`和`android:layout_height`对大多数组件来说都是必须的

* `match_parent`  
  填充至其父节点一样大
* `wrap_content`  
  根据包含内容需要改变大小

#### android:orientation

定义其子集组件是垂直排列还是水平排列

#### android:text

定义显示的文字。  
定义文字时并没有使用硬编码，而是使用了`string resource`。
将真正的文字内容定义在另一个文件中，然后在这里引用。这样可以方便地切换语种

### Creating string resources

每个项目都有一个默认的string file，叫`strings.xml`，位于`app/res/values`

修改`strings.xml`文件

```xml
<resources>
    <string name="app_name">GeoQuiz</string>
    <string name="question_text">
        Constantionople is the largest city in Turkey.
    </string>
    <string name="true_button">True</string>
    <string name="false_button">False</string>
</resources>
```

你可以任意命名你的string file。
只要你将其放在`app/res/values`中并且包含`resources`元素

## From Layout XML to View Objects

自动生成的`QuizActivity.java`文件

```java
package com.avalon.ash.geoquiz;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class QuizActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_quiz);
    }
}
```

当一个activity实例被创建时会自动调用其`onCreate`函数，
该函数中调用函数`public void setContentView(int layoutResID)`用以生成界面

### Resources and resource IDs

资源文件不属于代码文件，他们是一些图片，音频，视频或xml文件。  
我们可以通过resource ID获取到相应资源，比如`R.layout.activity_quiz`  
如果想查看ID确切的值，可以在`app/build/generated/source/r/debug`目录下
找到一个名为你package的文件夹，其中有一个`R.java`文件。  
该文件中定义了ID确切的值，切记该文件是自动生成的，不要手动改动该文件  
你看到的`R.java`并不一定是最新的，Android Studio并不会实时生成`R.java`  

Android会为layout文件和每个string生成一个ID，但不会为layout文件中的每一个widget生成ID。
因为不是所有的widget都需要。该程序中只有两个`Button`组件需要

为组件创建ID需要使用`android:id`属性

```xml
<Button
    android:id="@+id/true_button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/true_button"/>

<Button
    android:id="@+id/false_button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/false_button"/>
```

注意创建资源ID时需要使用`+`符号，只需要在第一次创建时使用该符号，创建后其它地方引用时就不需要添加

## Wiring Up Widgets

在`QuizActivity.java`中添加成员变量

```java
import android.widget.Button;

public class QuizActivity extends AppCompatActivity {
    private Button mTrueButton;
    private Button mFalseButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_quiz);
    }
}
```

如果不清楚应该引用什么包，可是使用快捷键`Alt + Enter`(or `Option + Return`)

### Getting references to widgets

可以通过这个函数在 Activity中获取到widget

```java
public View findViewById(int id);
```

在赋值给成员变量前记得先加`View`转化为`Button`

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_quiz);

    mTrueButton  = (Button)findViewById(R.id.true_button);
    mFalseButton = (Button)findViewById(R.id.false_button);
}
```

### Setting listeners

安卓程序是事件驱动的，所以要有`listener`来监听事件的发生

```java
mTrueButton = (Button)findViewById(R.id.true_button);
mTrueButton.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
        // Do nothing yet, but soon.
    }
});
```

## Making Toasts

toast 是给用户的一小段提示信息  

首先在`strings.xml`中添加文本资源

```xml
<string name="correct_toast">Correct!</string>
<string name="incorrect_toast">Incorrect!</string>
```

需要调用以下函数来创建 toast

```java
public static Toast makeText(Context context, int resId, int duration);
```
然后调用`Toast.show()`来显示 toast

```java
public void onClick(View v){
    Toast.makeText(QuizActivity.this,
            R.string.correct_toast,
            Toast.LENGTH_SHORT).show();
}
```
