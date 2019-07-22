---
layout: post
title:  Chapter 5 Your Second Activity
date:   2017-03-12 19:51:00 +0800
categories:
  - Reading
  - Android
---

## Setting Up a Second Activity

在`strings.xml`中添加新的字符串

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  ...
  <string name="warning_text">Are you sure want to do this?</string>
  <string name="show_answer_button">Show Answer</string>
</resources>
```

### Creating a new activity

创建一个`activity`需要用到至少三个文件：`java class file`, `xml layout`, `application manifest`。
为了避免出错最好使用`Android Studio`的向导创建

在`Android`视图模式下，右键`com.bignerdranch.android.geoquiz`，选择`New - Activity - Empty Activity`。
在弹出的对话框中，`Activity Name`填写`CheatActivity`，点击完成  
改写`activity_cheat.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"
              android:gravity="center"
              tools:context=".CheatActivity">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="24dp"
        android:text="@string/warning_text"/>
    <TextView
        android:id="@+id/answerTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="24dp"
        tools:text="Answer"/>
    <Button
        android:id="@+id/showAnswerButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/show_answer_button"/>
</LinearLayout>
```

### Declaring activity in the manifest

`manifest`文件位于`app/manifests`目录下，文件名为`AndroidManifest.xml`

### Adding a cheat button to QuizActivity

在`activity_quiz.xml`中添加`button`

```xml
<Button
  android:id="@+id/cheat_button"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="@string/cheat_button"/>
```

在`QuizActivity.java`中关联`button`

```java
private Button mCheatButton;

@Override
protected void onCreate(Bundle savedInstanceState) {
  mCheatButton = (Button)findViewById(R.id.cheat_button);
});
```

## Start an Activity

```java
public void startActivity(Intent intent)
```

该函数是系统的一个函数，更精确地说是`ActivityManager`

### Communicating with intent

`component`通过`intent`和系统通信。根据不同的通信要求`Intent`类有不同的构造函数。目前我们需要使用的构造函数为：

```java
public Intent(Context packageContext, Class<?> cls)
```

`Class<?>`参数即为需要让`ActivityManager`启动的`activity`，`Context`参数为包含该`activity`的`package`  
`QuizActivity.java`中，`mCheatButton`点击事件

```java
mCheatButton.setOnClickListener(new View.OnClickListener(){
  @Override
  public void onClick(View v){
    Intent i = new Intent(QuizActivity.this, CheatActivity.class);
    startActivity(i);
  }
})
```

在启动`activity`前，`ActivityManager`会线在`manifest`中查找要启动的`activity`的名字。

## Passing Data Between Activities

`QuizActivity`需要告诉`CheatActivity`当前问题的答案，`CheatActivity`需要告诉`QuizActivity`用户是否作弊

### Using intent extras

`Intent`的`extra`可以传递任意数据，采用键值对的形式。

在`CheatActivity.java`中

```java
public static Intent newIntent(Context packageContext, boolean answerIsTrue){
    Intent i = new Intent(packageContext, CheatActivity.class);
    i.putExtra(EXTRA_ANSWER_IS_TRUE, answerIsTrue);
    return i;
}
```

传递`QuizActivity.java`

```java
mCheatButton.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
        // start cheatActivity
        boolean answerIsTrue = mQuestionBank[mCurrentIndex].isAnswerTrue();
        Intent i = CheatActivity.newIntent(QuizActivity.this, answerIsTrue);
        startActivityForResult(i, REQUEST_CODE_CHEAT);
    }
});
```

获取`CheatActivity.java`

```java
mAnswerIsTrue = getIntent().getBooleanExtra(EXTRA_ANSWER_IS_TRUE, false);
```

显示答案`CheatActivity.java`

```java
mAnswerTextView = (TextView) findViewById(R.id.answerTextView);
mShowAnswer = (Button)findViewById(R.id.showAnswerButton);
mShowAnswer.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
        if(mAnswerIsTrue){
            mAnswerTextView.setText(R.string.true_button);
        }else {
            mAnswerTextView.setText(R.string.false_button);
        }
        setAnswerShowResult(true);
    }
});
```

### Getting a result back from a child activity

```java
public void startActivityForResult(Intent intent, int requestCode)
```

`requestCode`是用户自定义的一个整数，传给`child activity`后会再次被传回。
当启动多个`child activity`时可以用来区分是那个传回的。

在`child activity`中通过调用

```java
public final void setResult(int resultCode);
public final void setResult(int resultCode, Intent data);
```

来设定返回值

```java
private void setAnswerShowResult(boolean isAnswerShown){
    Intent data = new Intent();
    data.putExtra(EXTRA_ANSWER_SHOW, isAnswerShown);
    setResult(RESULT_OK, data);
}
```

当在`CheatActivity`按`返回`键时，`ActivityManager`会自动调用`QuizActivity`中的函数：

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data);
```

来获取返回值

在`CheatActivity.java`中添加函数以便于被获取

```java
public static boolean wasAnswerShown(Intent result) {
    return result.getBooleanExtra(EXTRA_ANSWER_SHOWN, false);
}
```

### Handling a result

添加变量保存返回结果

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    if(resultCode != Activity.RESULT_OK){
        return;
    }

    if(requestCode == REQUEST_CODE_CHEAT){
        if(data == null){
            return;
        }
        mIsCheater = CheatActivity.wasAnswerShow(data);
    }
}
```
