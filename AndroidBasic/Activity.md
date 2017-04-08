* [应用程序的国际化](#应用程序的国际化)
* [样式和主题](#样式和主题)
* [Activity启动方式](#activity启动方式)
	* [显示启动](#显示启动)
	* [隐式启动](#隐式启动)
* [数据传递](#数据传递)
	* [Intent传参](#intent传参)
	* [使用Bundle传参](#使用bundle传参)
	* [传List](#传list)
* [获取新界面的返回值](#获取新界面的返回值)

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

#### 传List

 1. 利用bundle传值

``` java
Intent intent = new Intent(MainActivity.this, ResultActivity.class);
Bundle bundle = new Bundle();
bundle.putSerializable("homeBeans", (Serializable) homeBeans);
intent.putExtras(bundle);
startActivity(intent);
```


 2. 获取方式
 
``` java
homeBeans = (List<HomeBean>) getIntent().getSerializableExtra("homeBeans");
```


### 获取新界面的返回值

 1. 启动Activity时候使用startActivityForResult取代startActivity
 
``` java
 public class MainActivity extends Activity {
 
    //请求码
    private static final int CHOOSE_ICON=0x001;
    ...
	
    public void chooseIcon(View v){
        Intent intent=new Intent(this,IconActivity.class);
        startActivityForResult(intent,CHOOSE_ICON);
    }
 }
```

 2. 在新的activity中返回值

``` java
public class IconActivity extends Activity {
    ...
    private void returnData(int res) {
        Intent data = new Intent();
        data.putExtra("icon", res);
        setResult(0, data);//第一个参数为resultCode
        finish();
    }
}
```

 3. 在原来的activity的onActivityResult中处理返回结果
 
``` java
public class MainActivity extends Activity {
    ...

    //获取新页面的结果
    //请求码requestCode： 用来判断到底是哪个页面返回的数据
    //结果码resultCode：一个新的页面可能有多种返回值的情况 原来的页面会根据不同的情况作出不同的处理
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (data != null) {
            if (requestCode == CHOOSE_ICON) {
                //可以取到头像
                int resId = data.getIntExtra("icon", 0);
                mIconIv.setImageResource(resId);
            }
            ....
        }
        super.onActivityResult(requestCode, resultCode, data);
    }
}
```


### 正常情况Activity的生命周期

 1. 七大回调方法
    -  onCreate：在活动第一次被创建的时候调用，应该在这个方法里面完成活动的初始化操作，比如加载布局，绑定事件等
    -  onRestart：Activity被重新启动的时候调用，如用户按Home开启其他活动然后再回到这个Activity的时候调用
    - onStart：表示当前Activity正在启动，但还没有出现在前台
    - onResume：表示Activity已经可见了，并且开始出现在前台
    - onPause：表示Activity正在停止，正常情况下，紧接着onStop马上就会调用，通常在这个方法里面释放一些系统资源，但是不能太耗时
    - onStop：表示Activity即将停止，可以稍微做点重量级的回收工作，同样不能太耗时
    - onDestrroy：表示Activity即将被销毁，这是生命周期最后一个回调

 
 2. 生命周期图

![enter description here][1]
 
 3. 几种具体的情况
	-  针对一个特点的Activity，第一次启动，回调如下 ：onCreate->onStart->onResume
	- 当用户打开新的Activity或者切换桌面的时候，回调如下：onPause->onStop
	- 当用户再次返回原Activity，回调如下：onRestart->onStart-onResume
	- 当用户按back键返回的时候，回调如下：onPause->onStop->onDestroy
	- 当Activity被系统回收后再次打开，回调过程：和1一样，但是只是生命周期方法一样，有过程不一样

 4. 从整个生命周期来看
    - onCreate和onDestory是对称的，标志Activity的创建和销毁
    - onStart和onStop是对称的，代表程序是否可见
    - onResume和onPause是对称的，标志程序的是否在前台
    - onStart和onResume，onPause和onStop的区别：前者只是在可见，而后者代表是否在前台，如对话框
    - 开启新的Activity，旧的onPause和新的onResume谁先执行：从源码角度来看，旧的onPause先执行


  [1]: ./images/Activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png "Activity生命周期"