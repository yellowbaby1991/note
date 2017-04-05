### 应用程序的国际化

 1. 使用@string配置取代layout中的硬编码
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/text" />
    
</RelativeLayout>

```

 2. values和values-en文件夹下建立对应的string.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">活动</string>
    <string name="text">你好</string>
</resources>
```

``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">activity</string>
    <string name="text">hello</string>
</resources>
```

### 样式和主题

 1. 在values/styles.xml中创建样式
 
``` xml
<resources xmlns:android="http://schemas.android.com/apk/res/android">

    <!--
        Base application theme, dependent on API level. This theme is replaced
        by AppBaseTheme from res/values-vXX/styles.xml on newer devices.
    -->
    <style name="AppBaseTheme" parent="android:Theme.Light">
        <!--
            Theme customizations available in newer API levels can go in
            res/values-vXX/styles.xml, while customizations related to
            backward-compatibility can go here.
        -->
    </style>

    <!-- Application theme. -->
    <style name="AppTheme" parent="AppBaseTheme">

    </style>
    
    <style name="simple_tv_style">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">40dp</item>
        <item name="android:gravity">center_vertical</item>
    </style>

    <style name="tv_style" parent="@style/simple_tv_style">
        <item name="android:paddingLeft">10dp</item>
        <item name="android:textSize">18sp</item>
        <item name="android:textColor">#0F0</item>
        <item name="android:textStyle">bold</item>
    </style>
    
    <style name="page_style">
        <item name="android:background">#F00</item>
    </style>
    
    
</resources>
```


 2. 在layout中引用样式
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"   >
    ...
    <TextView
        style="@style/tv_style"   
        android:text="装逼模式" />
    
    <TextView 
        style="@style/simple_tv_style"
        android:text="其他设置" />
    
</LinearLayout>
```

 3. 在ManifAndroidManifest.xml中引用主题
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.m520it.style"
    android:versionCode="1"
    android:versionName="1.0" >

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        ...
    </application>

</manifest>
```

### Activity的启动

 1. 显示启动

``` java
Intent intent=new Intent(MainActivity.this, SecondActivity.class); 
startActivity(intent);
```


 2. 隐式启动