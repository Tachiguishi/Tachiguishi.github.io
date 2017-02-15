---
layout: post
title:  Chapter 2 Android and Model-View-Controller
date:   2017-02-14 22:06:00 +0800
categories:
  - Reading
  - Android_Programming_by_Chris_Stewart
---

将上一张程序改为有多个问答题目

## 创建`Question`类

在`geoquiz`包中添加`Question`类，右键`New - Java Class`

```java
package com.avalon.ash.geoquiz;

public class Question{
  private int mTextResId;
  private boolean mAnswerTrue;

  public Question(int textResId, boolean answer){
    mTextResId = textResId;
    mAnswerTrue = answer;
  }
}
```

`Question`类中包含问题内容和答案。但是问题不是直接保存在类中，而是只保存问题文本的资源id

### 自动生成`getter`和`setter`

`File - Settings`打开配置对话框。`Editor - Code Style - Java`。在右侧窗口中选择`Code Generation`标签选项  
选中`Inner classes`，在`Field / Name prefix`中填写`m`，`Static field / Name prefix`中填写`s`。点击`OK`结束配置

在打开的`Question.java`文件中，在构造函数后面鼠标右键，
点击`Generate`，选择`Getter and Setter`, 然后选择内部变量`mTextResId`和`mAnswerTrue`，点击生成  
![code generation](https://i.imgur.com/awvXPuO.png)

## Android and Model-View-Controller

* model layer 包含 model 对象，通常对应一个物理类，用于存储和管理物理数据
* view layer 用于定义用户界面并响应用户指令
* controller layer用于连接 model 和 view。管理业务逻辑。在Android中它通常是`Activity`, `Fragment`, `Service`的子类

## update the view layer

`activity_quiz.xml`中添加`next_button`，修改显示问题的`TextView`

```xml
<TextView
  android:id="@+id/question_text_view"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:padding="24dp"/>

<Button
  android:id="@+id/next_button"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="@string/next_button"/>
```

`strings.xml`中添加相关字符串

```xml
<string name="next_button">Next</string>
<string name="question_oceans">Question 1</string>
<string name="question_mideast">Question 2</string>
<string name="question_african">Question 3</string>
```

## update the controller layer

`QuizActivity.java`中添加变量，并绑定显示问题的`TextView`和`Next`按钮

```java
public class QuizActivity extends AppCompatActivity{
  private Button mTrueButton;
  private Button mFalseButton;
  private Button mNextButton;
  private TextView mQuestionTextView;

  private Question[] mQuestionBank = new Question[]{
    new Question(R.string.question_oceans, true),
    new Question(R.string.question_mideast, false),
    new Question(R.string.question_african, false),
  };

  private int mCurrentIndex = 0;

  private void updateQuestion(){
    int question = mQuestionBank[mCurrentIndex].getTextResID();
    mQuestionTextView.setText(question);
  }

  @Override
  protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_quiz);

    mQuestionTextView = (TextView)findViewById(R.id.question_text_view);
    updateQuestion();

    mNextButton = (Button)findViewById(R.id.next_button);
    mNextButton.setOnClickListener(new View.OnClickListener(){
        @Override
        public void onClick(View v){
            mCurrentIndex = (mCurrentIndex + 1) % mQuestionBank.length;
            updateQuestion();
        }
    });

    // ...
  }
}
```

增加答案校验功能

```java
private void checkAnswer(boolean userPressedTrue){
    boolean answerIsTrue = mQuestionBank[mCurrentIndex].isAnswerTrue();

    int messageResId = 0;
    if(userPressedTrue == answerIsTrue){
        messageResId = R.string.correct_toast;
    }else{
        messageResId = R.string.incorrect_toast;
    }

    Toast.makeText(this, messageResId, Toast.LENGTH_SHORT).show();
}

@Override
protected void onCreate(Bundle savedInstanceState){
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_quiz);

  // ...

  mTrueButton = (Button)findViewById(R.id.true_button);
  mTrueButton.setOnClickListener(new View.OnClickListener(){
      @Override
      public void onClick(View v){
          checkAnswer(true);
      }
  });
  mFalseButton = (Button)findViewById(R.id.false_button);
  mFalseButton.setOnClickListener(new View.OnClickListener(){
      @Override
      public  void onClick(View v){
          checkAnswer(false);
      }
  });
}
```

## 在硬件设备上运行

首先配置手机，打开`开发者选项`中的`USB调试`功能
(`Settings - Developer options - USB debugging`)  

使用数据线连接手机与电脑(Mac上自动连接，而Windows上需要[安装驱动](https://developer.android.google.cn/studio/run/oem-usb.html)  
如果驱动为自动安装则需要手动安装，在`计算机 - 设备管理`中可以看到有个`其它设备`-`ADB Interface`。  
![abd driver uninstalled](https://i.imgur.com/cBt3Mkw.png)

`右键 - 更新驱动程序`，在弹出的对话框中手动选择驱动所在目录，然后点击安装即可  
安装成功后可以看到`Android Device - Android Composite ADB Interface`  
![abd driver installed](https://i.imgur.com/jbBol81.png)

如果连接成功可以在`Android Studio`的`Terminal`中运行`adb devices`进行检测  
运行目录为`$(sdk/platform-tools)`，该命令可以列出所有已连接设备  
![adb devices](https://i.imgur.com/oLV8nQ7.png)

运行程序，选择你所连接的硬件设备而非模拟器，则可以你所连接的设备上测试你的程序。  
之后就可以`Android monitor`中看到你设备的状态  
![android monitor](https://i.imgur.com/WSuJl6u.png)
