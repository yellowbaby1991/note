#### 概述
　　Material design 是一种达到可视化，交互性，动效以及多屏幕适应的全面设计，Google为了方便开发者推出了兼容5.0前后的Material design支持库，下面是一些常用的控件
  
#### Toolbar
----------

#####  基本用法

 1. Toolbar的出现是为了取代原有的ActionBar，所以想使用Toolbar首先需要将原有的ActionBar隐藏，将style改成NoActionBar
    　
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="yellow.com.materialdesign">

    <application
        ...
        android:theme="@style/AppTheme">
        ...
    </application>

</manifest>
```

``` xml
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.NoActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>
```

 2. 将Toolbar放入布局文件中
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary" //背景
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" //弹出的背景色 />
    </FrameLayout>
    
</RelativeLayout>

```

 3. Activity中设置一下

``` java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);//这样toolbar就可以正常使用了
	}
```



##### Toolbar添加文字

 1. 在activity中设置title和subtitle

``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        toolbar.setTitle("主标题");
        toolbar.setSubtitle("副标题");
        setSupportActionBar(toolbar);
    }
}
```


 2. 在AndroidManifest.xml中设置label属性

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="yellow.com.materialdesigndemo">
    ...
    <application
        android:label="Fruit"
        android:theme="@style/AppTheme">
		...
    </application>

</manifest>
```

##### 给Toolbar添加按钮

 1. 创建menu/toobar.xml文件，除了设置背景图和文字外，还可以指定showAsAction属性，showAsAction属性含义如下：
	- always：表示永远显示在屏幕
	- ifRoom：表示屏幕空间足够就显示，不够就在菜单中
	- never：表示永远在菜单中
``` xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/backup"
        android:icon="@drawable/ic_backup"
        android:title="Backup"
        app:showAsAction="always"></item>
    <item
        android:id="@+id/delete"
        android:icon="@drawable/ic_delete"
        android:title="Backup"
        app:showAsAction="ifRoom"></item>
    <item
        android:id="@+id/settings"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never"></item>
</menu>
```


 2. 在onCreateOptionsMenu中添加菜单

``` stylus
public class MainActivity extends AppCompatActivity {
    ...
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.toolbar, menu);//加载菜单
        return true;
    }
	...
}
```
 1. 重写onOptionsItemSelected处理点击事件
``` stylus
public class MainActivity extends AppCompatActivity {
    ...
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
                mDrawerLayout.openDrawer(GravityCompat.START);
                break;
            case R.id.backup:
                Toast.makeText(this, "backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this, "delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.settings:
                Toast.makeText(this, "settings", Toast.LENGTH_SHORT).show();
                break;
        }
        return true;
    }
	...
}
```
##### 给Toolbar添加导航按钮
 1. 在MainActivity中打开导航属性
``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        ActionBar actionBar = getSupportActionBar();//得到Toolbar
        if (actionBar != null){
            actionBar.setDisplayHomeAsUpEnabled(true);//打开开关
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);//设置图片
        }

    }
}

```
 2. 在onOptionsItemSelected中处理点击事件

``` java
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;
	
	@Override
    protected void onCreate(Bundle savedInstanceState) {
		...
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
		...
    }
	
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
                mDrawerLayout.openDrawer(GravityCompat.START);
                break;
            ...
        }
        return true;
    }
```

#### DrawerLayout
----------
##### 基本用法

 1. DrawerLayout是一个布局，允许放入两个直接子控件，第一个是内容，第二个是滑动菜单，因此布局如下

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    //内容
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    </FrameLayout>

	//菜单
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="菜单栏" />

</android.support.v4.widget.DrawerLayout>

```

 2. 给菜单栏设置layout_gravity属性，left表示菜单在左，right表示菜单在右，start根据系统语言自行判断

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    ...
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:text="菜单栏"
        android:textSize="30sp"
        android:background="#FFF"/>

</android.support.v4.widget.DrawerLayout>
```



##### 使用NavigationView打造菜单栏

 1. 添加NavigationView需要的design依赖和实现圆形图片的CircleImageView依赖

``` xml
dependencies {
    ...
    compile 'com.android.support:design:24.2.1'
    compile 'de.hdodenhof:circleimageview:2.1.0'
}
```

 2. NavigationView由两个部分组成，菜单栏和菜单头组成，先定义菜单
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_call"
            android:icon="@drawable/nav_call"
            android:title="Call"></item>
        <item
            android:id="@+id/nav_friends"
            android:icon="@drawable/nav_friends"
            android:title="Friends"></item>
        <item
            android:id="@+id/nav_location"
            android:icon="@drawable/nav_location"
            android:title="Location"></item>
        <item
            android:id="@+id/nav_mail"
            android:icon="@drawable/nav_call"
            android:title="Mail"></item>
        <item
            android:id="@+id/nav_task"
            android:icon="@drawable/nav_task"
            android:title="Tasks"></item>
    </group>
</menu>
```

 3. 使用CircleImageView定义有圆形头像的菜单头部

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:background="?attr/colorPrimary"
    android:padding="10dp">
     <!--圆形头像-->
    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/icon_image"
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:layout_centerInParent="true"
        android:src="@drawable/nav_icon" />
     <!--邮箱-->
    <TextView
        android:id="@+id/username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:text="306790935@qq.com"
        android:textColor="#FFF"
        android:textSize="14sp" />
     <!--昵称-->
    <TextView
        android:id="@+id/mail"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/username"
        android:text="Yellow Baby"
        android:textColor="#FFF"
        android:textSize="14sp" />

</RelativeLayout>
```

4. 使用NavigationView关联菜单栏和菜单头取代之前简易的TextView

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    </FrameLayout>

    <android.support.design.widget.NavigationView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header"
        android:layout_gravity="start">
    </android.support.design.widget.NavigationView>

</android.support.v4.widget.DrawerLayout>

```


