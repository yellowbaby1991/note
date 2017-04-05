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

### Activity启动方式

#### 显示启动

``` java
Intent intent=new Intent(MainActivity.this, SecondActivity.class); 
startActivity(intent);
```


#### 隐式启动
 
``` java
//匹配规则如下
//action：在Intent中只可以设置一个action的值，如果manifest中配置了Activity的action，那么在Intent中这个值久必须要有，如果manifest中有多个action的值，在Intent中任意配置一个就可以
//category：Intent中可以包含多个category，但每个category必须和manifest中的其中一个category相同，即：Intent中包含的category必须是manifest中的category的子集
//data：同action类似，注意setDataAndType()方法
Intent intent=new Intent("MyIntent");
intent.addCategory("hello");
startActivity(intent);
```

``` xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="app.yellow.activity">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
       ...
        <activity android:name=".SecondActivity">
            <intent-filter>
                <action android:name="MyIntent"></action>
                <category android:name="hello" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
    </application>

</manifest>

 ```
### 数据传递
#### Intent传参

 1. 使用putExtra把参数压入intent

``` java
Intent intent=new Intent(this, ThirdActivity.class);
//需要将对象传给新的页面
Student student = new Student("lisi", 20);
intent.putExtra("student", student);//该对象必须可被序列化
startActivity(intent);
```

2. 获取方式

``` java
Intent intent = getIntent();
Student student = (Student) intent.getSerializableExtra("student");
```

#### 使用Bundle传参

 1. Bundle可以用来IPC通信，也经常用于四大组件的传参，当数据较多的时候可以选择使用Bundle一次性封装传递过去，如果更复杂的话就自己封装一个对象，当然也要可以被序列化的
 
 2. 使用Bundle封装数据

``` java
String name="zhangsan";
Bundle bundle=new Bundle();
bundle.putString("name", name);
bundle.putInt("age", 18);
//将Bundle对象与intent对象关联
intent.putExtra("person", bundle);
startActivity(intent);
```

 3. 获取数据

``` java
Bundle bundleExtra = intent.getBundleExtra("person");
String name = bundleExtra.getString("name");
int age = bundleExtra.getInt("age");
```

### 获取新界面的返回值