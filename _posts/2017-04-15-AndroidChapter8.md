---
layout: post
title:  Chapter 8 Creating User Interfaces with Layouts and Widgets
date:   2017-04-15 06:15:00 +0800
categories:
  - Reading
  - Android Programming by Chris Stewart
---

## Upgrading Crime

添加属性

```java
public class Crime{
  private UUID mId;
  private String mTitle;
  private Date mDate;
  private boolean mSolved;

  public Crime(){
    mId = UUID.randomUUID();
    mDate = new Date();
  }
}
```

## Upgrading the Layout

`fragment_crime.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:orientation="vertical"
  >
  <TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@string/crime_title_label"
    style="?android:listSeparatorTextViewStyle"
    />
  <EditText android:id="@+id/crime_title"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginLeft="16dp"
    android:layout_marginRight="16dp"
    android:hint="@string/crime_title_hint"
    />
  <TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@string/crime_details_label"
    style="?android:listSeparatorTextViewStyle"
    />
  <Button android:id="@+id/crime_date"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginLeft="16dp"
    android:layout_marginRight="16dp"
    />
  <CheckBox android:id="@+id/crime_solved"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginLeft="16dp"
    android:layout_marginRight="16dp"
    android:text="@string/crime_solved_label"
    />
</LinearLayout>
```

## Wiring Widgets

在`CrimeFragment`中绑定`crime_date`和`crime_solved`控件

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup parent,
        Bundle savedInstanceState) {
    View v = inflater.inflate(R.layout.fragment_crime, parent, false);

    // ...

    mDateButton = (Button)v.findViewById(R.id.crime_date);
    mDateButton.setText(mCrime.getDate().toString());
    mDateButton.setEnabled(false);

    mSolvedCheckBox = (CheckBox)v.findViewById(R.id.crime_solved);
    mSolvedCheckBox.setOnCheckedChangeListener(new OnCheckedChangeListener() {
        @Override
        public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
            // Set the crime's solved property
            mCrime.setSolved(isChecked);
        }
    });

    return v;
}
```
