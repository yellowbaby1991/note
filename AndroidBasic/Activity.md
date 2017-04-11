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

### Activity的启动方式

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
### Activity的启动模式

 1. 四种启动模式
	 - standard：默认的活动启动模式，系统不会在乎这个活动是否已经在返回栈中存在，每次启动都会创建一个新的活动实例
	 - singleTop：当启动活动的时候发现返回栈的栈顶已经是该活动，则认为可以直接使用它，不会再创建新的活动实例，需要注意的是，这个Activity的不会调用onCreate和onStart，会调用onNewIntent，如果活动不在栈顶，还是会创建新实例
	 - singleTask：每次启动活动时，先判断该Activity是否在栈内，如果不存在则创建，如果存在，就判断该Activity上面是否有其他Activity，如果有，则关闭其他的Activity，也就是cleartop
	 - singleInstance：会有一个单独的返回栈来管理这个Activity，不管哪个应用程序来访问这个Activity，都公用的同一个返回栈

 2. 四种启动模式的应用场景
	 - singleTop：新闻应用的内容栏，假如收到了10个新闻推送，连续打开10个内容界面是噩梦的，使用该模式可以只在栈顶创建一个唯一的活动
	 - singleTask：适合做程序的入口，如浏览器的主页，不管从多少个应用启动浏览器，只会启动一次，其他情况都走onNewIntent，并且清空主界面上的其他页面
	 - 

 3. 在隐式启动的时候指定启动模式
 ```java
//1. FLAG_ACTIVITY_NEW_TASK 指定singleTask模式
//2. FLAG_ACTIVITY_SINGLE_TOP 指定singleTop模式
//3. FLAG_ACTIVITY_CLEAR_TOP 和singleTask配合使用，clear top
//4. FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS 等同于android:excludeFromRecents="true"，具有这个标记的activity不会出现在历史列表中 
Intent intent = new Intent("android.intent.action.APP_A_SECOND_ACTIVITY");
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

 4. 在AndroidMenifest.xml中指定启动模式

``` xml
  <activity
	android:name=".SecondActivity"
	android:label="@string/title_activity_second"
	android:launchMode="singleTop"
	android:theme="@style/AppTheme.NoActionBar" />
```


### 数据传递
#### Intent传参

　使用intent传参步骤如下
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

　Intent可传递的数据类型

 1. 8种基本数据类型及其数组
 2. String（String实现了Serializable）/CharSequence实例类型的数据及其数组 
 3. 实现了Parcelable的对象及其数组 (自己来做, 操作较复杂, 但速度快) 
 4. 实现了Serializable的对象及其数组(系统来做, 操作简单, 但速度慢)  


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


  
### 异常情况Activity的生命周期

 1. 资源相关的系统配置发生改变导致Activity被杀死并重新创建，经典例子就是横屏，重建过程如下：
    - 当系统配置发生变化，Activity会被销毁，其onPause，onStop，onDestory均被调用
    -  同时因为Activity是被意外终止的，所以系统会调用onSaveInstanceState方法保存当前Activity的状态，正常情况系统不会调用这个方法
    -  当Activity被重新创建后，系统调用onRestoreInstanceState，并且把onSaveInstanceState方法所保存的Bundle对象作为参数传递给onRestoreInstanceState和onCreate方法
    -  因此可以通过onRestoreInstanceState和OnCreate方法判断Activity是否被重建了，然后取出数据恢复，时序上来说，onRestoreInstanceState在onStart后

 2. 重建流程图
 
 ![enter description here][2]
 
 
 3. 示例代码如下

``` java
public class MainActivity extends AppCompatActivity {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        if (null != savedInstanceState) {//如果在oncreate中恢复数据需要判断savedInstanceState是否为null
            int IntTest = savedInstanceState.getInt("IntTest");
            String StrTest = savedInstanceState.getString("StrTest");
        }
        setContentView(R.layout.activity_main);

    }

    /**
     * 在横屏的时候调用该方法存储数据
     */
    @Override
    public void onSaveInstanceState(Bundle savedInstanceState) {
        savedInstanceState.putInt("IntTest", 1);
        savedInstanceState.putString("StrTest", "savedInstanceState test");
        super.onSaveInstanceState(savedInstanceState);
    }

    /**
     * 在重新创建Activity后执行，在onCreate之后
     */
    @Override
    public void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        int mCount = savedInstanceState.getInt("IntTest");
        String StrTest = savedInstanceState.getString("StrTest");
    }

}
```

 4. 如果不想Activity被横屏重建，可以设置configChanges属性
 
``` xml
 <activity android:name=".MainActivity" android:configChanges="orientation" >
```

### 任务栈

 1. Android使用任务栈来管理Activity，一个任务就是一组存在栈里活动Activity的集合
 2. 当我们启动一个新活动，会在任务栈中压入并且处于栈顶的位置
 3. 当我们按下back或者调用finish方法的时候，栈顶的Activity就会出栈


  [1]: ./images/Activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png "Activity生命周期"
  [2]: ./images/%E9%87%8D%E5%BB%BA%E6%B5%81%E7%A8%8B%E5%9B%BE.png "重建流程图"