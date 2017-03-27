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
 3. 重写onOptionsItemSelected处理点击事件
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


