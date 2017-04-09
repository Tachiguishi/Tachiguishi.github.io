---
layout: post
title:  Chapter 7 UI Fragments and the Fragments Manager
date:   2017-04-09 14:02:00 +0800
categories:
  - Reading
  - Android Programming by Chris Stewart
---

从本章开始我们要做一个`CriminalIntent`应用。为了界面的灵活性，需要使用`fragment`

## Starting CriminalIntent

### Adding dependencies in Android Studio

在`app/build.gradle`中添加依赖项。注意，项目有两个`build.gradle`文件，
一个是针对整个项目的，一个是针对`app module`。我们要更改的是后一个。

直接添加依赖项容易出错，
但可以通过`File - Project Structure - Modules - app - Dependencies`向导添加。
我们需要添加的是`Library Dependency`中的`support-v4`  
添加完成后你会在`app/build.gradle`文件中看到类似语句

```xml
dependencies{
  compile fileTree(include: ['*.jar'], dir: 'libs')
  compile 'com.android.support:support-v4:25.0.1'
}
```

### Creating the Crime class

```java
public class Crime{
  private UUID mId;
  private String mTitle;

  public Crime(){
    mId = UUID.randomUUID();
  }
}
```

## Hosting a UI Fragment

为了管理`UI fragment`，`activity`必须：  
* 在`layout`为其定义一个展示的`spot`
* 管理`fragment`的生命周期

### The fragment lifecycle

![fragment lifecycle](https://imgur.com/gM92WqT)

### Two approaches to hosting

* add the fragment to the activity's layout
* add the fragment in the activity's layout

第一种方式简单但缺乏灵活性，第二种方式更复杂当可以随时更改界面样式

### Defining a container view

`activity_crime.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:id="@+id/fragment_container"
  android:layout_width="match_parent"
  android:layout_height="match_parent"/>
```

## Creating a UI Fragment

### Defining CrimeFragment's layout

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical"
  >
  <EditText android:id="@+id/crime_title"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="@string/crime_title_hint"
    />
</LinearLayout>
```

### Creating the CrimeFragment class

新建`CrimeFragment.java`

```java
import android.support.v4.app.Fragment;
public class CrimeFragment extends Fragment {
}
```

注意`Fragment`有两个库可以引用，`android.app`和`android.support.v4.app`，请选择后者

#### Implementing fragment lifecycle methods

实现`onCreate()`函数

```java
public class CrimeFragment extends Fragment {
    private Crime mCrime;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mCrime = new Crime();
    }
}
```

注意和`activity`不同的是`fragment`的所有生命周期函数都是`public`，
且并不是在`onCreate()`绑定`layout`文件，而是需要通过`onCreateView()`函数实现

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle savedInstanceState) {
    View v = inflater.inflate(R.layout.fragment_crime, container, false);
    return v;
}
```

#### Wiring widgets in a fragment

在`onCreateView()`函数中绑定`widget`

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle savedInstanceState) {
    View v = inflater.inflate(R.layout.fragment_crime, container, false);
    mTitleField = (EditText)v.findViewById(R.id.crime_title);
    mTitleField.addTextChangedListener(new TextWatcher() {
        @Override
        public void beforeTextChanged(
            CharSequence s, int start, int count, int after) {
            // This space intentionally left blank
        }
        @Override
        public void onTextChanged(
            CharSequence s, int start, int before, int count) {
            mCrime.setTitle(s.toString());
        }
        @Override
        public void afterTextChanged(Editable s) {
            // This one too
        }
    });

    return v;
}
```

## Adding a UI Fragment to the FragmentManager

我们使用`FragmentManager`来操作管理`fragment`

```java
public class CrimeActivity extends FragmentActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime);
        FragmentManager fm = getSupportFragmentManager();
    }
}
```

### Fragment transactions

```java
public class CrimeActivity extends FragmentActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime);
        FragmentManager fm = getSupportFragmentManager();
        Fragment fragment = fm.findFragmentById(R.id.fragment_container);
        if (fragment == null) {
            fragment = new CrimeFragment();
            fm.beginTransaction()
                .add(R.id.fragment_container, fragment)
                .commit();
        }
    }
}
```
