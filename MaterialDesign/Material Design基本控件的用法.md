1. [概述](#概述)
2. [Toolbar](#toolbar)
	-  [基本用法](#基本用法)
	-  [Toolbar添加文字](#toolbar添加文字)
	- [给Toolbar添加按钮](#给toolbar添加按钮)
	- [给Toolbar添加导航按钮](#给toolbar添加导航按钮)
3. [DrawerLayout](#drawerlayout)
	- [基本用法](#基本用法)
	-  [使用NavigationView打造菜单栏](#使用navigationview打造菜单栏)
4. [FloatingActionButton](#floatingactionbutton)
5. [Snackbar](#snackbar)
6. [CardView+RecyclerView](#cardviewrecyclerview)
7. [SwipeRefreshLayout](#swiperefreshlayout)
8. [CoordinatorLayout](#coordinatorlayout)
	-  [基本作用](#基本作用)
	-  [CoordinatorLayout与FloatingActionButton](#coordinatorlayout与floatingactionbutton)
	-  [CoordinatorLayout与AppBarLayout](#coordinatorlayout与appbarlayout)
	- [CoordinatorLayout与CollapsingToolbarLayout](#coordinatorlayout与collapsingtoolbarlayout)
9. [TextInputLayout](#textinputlayout)
10. [TabLayout](#tablayout)

#### 概述
　　Material design 是一种达到可视化，交互性，动效以及多屏幕适应的全面设计，Google为了方便开发者推出了兼容5.0前后的Material design支持库，下面是一些常用的控件
  
#### Toolbar
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



##### 添加Home键拖出menu

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
        app:menu="@menu/nav_menu"  //关联菜单栏
        app:headerLayout="@layout/nav_header" //关联菜单头
        android:layout_gravity="start">
    </android.support.design.widget.NavigationView>

</android.support.v4.widget.DrawerLayout>

```

#### FloatingActionButton

 1. 使用FloatingActionButton添加悬浮按钮

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

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:elevation="8dp" 
            android:src="@drawable/ic_done" />
    </FrameLayout>
	
     ...<!--菜单栏-->

</android.support.v4.widget.DrawerLayout>

```

 2. 添加监听事件，和Button类似

``` java
    protected void onCreate(Bundle savedInstanceState) {
		...
        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                 Toast.makeText(MainActivity.this, "Data restored", Toast.LENGTH_SHORT).show();
            }
        });
	}
```



#### Snackbar

 1.  Snackbars是一种比Toast更先进的提示工具，特点是加入了一个可以交互的按钮，比如删除的时候，可以提供一个撤回按钮，用法和Toast类似
 
``` java
    protected void onCreate(Bundle savedInstanceState) {
		...
        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Data deleted", Snackbar.LENGTH_SHORT).show();
            }
        });
	}
```
 2.  添加交互按钮监听

 ``` java
    protected void onCreate(Bundle savedInstanceState) {
		...
        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Data deleted", Snackbar.LENGTH_SHORT).setAction("撤销", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Toast.makeText(MainActivity.this, "Data restored", Toast.LENGTH_SHORT).show();
                    }
                }).show();
            }
        });
	}
```

#### CardView+RecyclerView

 1. 使用CardView+RecyclerView可以做出很漂亮的卡片布局效果，首先添加依赖

``` xml
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:24.2.1'
    compile 'com.android.support:cardview-v7:24.2.1'
	compile 'com.github.bumptech.glide:glide:3.7.0' //图片加载框架
}
```

 2. 把RecycleView加入内容布局

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:elevation="8dp"
            android:src="@drawable/ic_done" />
    </android.support.design.widget.CoordinatorLayout>

    <!--菜单栏-->
    ...

</android.support.v4.widget.DrawerLayout>

```

 3. 使用CardView创建RecyclerView需要的item布局

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp"//圆角弧度>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/fruit_image"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop" />

        <TextView
            android:id="@+id/fruit_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp" />
    </LinearLayout>

</android.support.v7.widget.CardView>
```

 4. 创建需要的RecyclerView.Adapter

``` java
public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {

    private Context mContext;

    private List<Fruit> mFruitList;

    public FruitAdapter(List<Fruit> fruitList) {
        mFruitList = fruitList;
    }

    static class ViewHolder extends RecyclerView.ViewHolder {
        CardView cardView;
        ImageView fruitImage;
        TextView fruitName;

        public ViewHolder(View view) {
            super(view);
            cardView = (CardView) itemView;
            fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
            fruitName = (TextView) view.findViewById(R.id.fruit_name);
        }
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (mContext == null) {
            mContext = parent.getContext();
        }
        View view = LayoutInflater.from(mContext).inflate(R.layout.fruit_item, parent, false);
        final ViewHolder holder = new ViewHolder(view);
        return holder;
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        Fruit fruit = mFruitList.get(position);
        holder.fruitName.setText(fruit.getName());
		//使用Glide加载高精度的图片，防止卡死
        Glide.with(mContext).load(fruit.getImageId()).into(holder.fruitImage);
    }

    @Override
    public int getItemCount() {
        return mFruitList.size();
    }
}

```

5. 给RecyclerView设置数据源，布局管理器等

``` java
public class MainActivity extends AppCompatActivity {

    private FruitAdapter adapter;
	
	private List<Fruit> fruitList = new ArrayList<>();
	
    private Fruit[] fruits = {new Fruit("Apple", R.drawable.apple), new Fruit("Banana", R.drawable.banana),
            new Fruit("Orange", R.drawable.orange), new Fruit("Watermelon", R.drawable.watermelon),
            new Fruit("Pear", R.drawable.pear), new Fruit("Grape", R.drawable.grape),
            new Fruit("Pineapple", R.drawable.pineapple), new Fruit("Strawberry", R.drawable.strawberry),
            new Fruit("Cherry", R.drawable.cherry), new Fruit("Mango", R.drawable.mango)};
			
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
	    ...
        initFruits();
		
        RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
        GridLayoutManager layoutManager = new GridLayoutManager(this, 2);
        recyclerView.setLayoutManager(layoutManager);
        adapter = new FruitAdapter(fruitList);
        recyclerView.setAdapter(adapter);
    }
	
    private void initFruits() {
        fruitList.clear();
        for (int i = 0; i < 50; i++) {
            Random random = new Random();
            int index = random.nextInt(fruits.length);
            fruitList.add(fruits[index]);
        }
    }
}
```



#### SwipeRefreshLayout

 1. 使用SwipeRefreshLayout实现下拉刷新效果，先把RecyclerView包裹起来

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
         ... <!--标题栏-->
 
        <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/swipe_refresh"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

            <android.support.v7.widget.RecyclerView
                android:id="@+id/recycler_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"/>
        </android.support.v4.widget.SwipeRefreshLayout>

    ... <!--浮动按钮-->
    </android.support.design.widget.CoordinatorLayout>

  
    ... <!--菜单栏-->

</android.support.v4.widget.DrawerLayout>

```


 2. MainActivity中处理下拉事件

``` java
public class MainActivity extends AppCompatActivity {

    private SwipeRefreshLayout swipeRefresh;
	
	    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		...
        swipeRefresh = (SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        swipeRefresh.setColorSchemeResources(R.color.colorPrimary);
        swipeRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                refreshFruits();
            }
        });		
	}
	...
    private void refreshFruits() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        initFruits();
                        adapter.notifyDataSetChanged();
                        swipeRefresh.setRefreshing(false);
                    }
                });
            }
        }).start();
    }	
}
```



#### CoordinatorLayout
##### 基本作用
 1. 作为顶层布局
 2. 调度协调布局
##### CoordinatorLayout与FloatingActionButton
 1. 使用CoordinatorLayout让FloatingActionButton不会被Snackbar遮挡，将之前的FrameLayout替换即可

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:elevation="8dp"
            android:src="@drawable/ic_done" />
    </android.support.design.widget.CoordinatorLayout>

    <!--菜单栏-->
    ...

</android.support.v4.widget.DrawerLayout>

```





##### CoordinatorLayout与AppBarLayout

 1. 使用AppBarLayout可以防止CoordinatorLayout内标题和内容重叠
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.AppBarLayout //使用AppBarLayout包裹
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
        </android.support.design.widget.AppBarLayout>

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior" //设置表现形式
            android:layout_height="match_parent" />
	...
    </android.support.design.widget.CoordinatorLayout>

    <!--菜单栏-->
    ...

</android.support.v4.widget.DrawerLayout>

```


 2. 设置Toolbar的layout_scrollFlags属性来实现联动效果

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
                app:layout_scrollFlags="scroll|enterAlways|snap"
				//srcoll表示上滚的时候标题会被隐藏
				//enterAlways表示当下滚的时候标题会重新显示
				//snap表示当标题还没有完全隐藏的时候会根据当前的距离自动隐藏
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
        </android.support.design.widget.AppBarLayout>

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior"
            android:layout_height="match_parent" />

        ...
		  <!--浮动按钮-->
    </android.support.design.widget.CoordinatorLayout>

    <!--菜单栏-->
    ...

</android.support.v4.widget.DrawerLayout>

```

##### CoordinatorLayout与CollapsingToolbarLayout

 1. CollapsingToolbarLayout可以实现可折叠式标题栏，但是只能作为AppBarLayout的直接子布局来使用，所以布局如下

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp"
        android:fitsSystemWindows="true">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="?attr/colorPrimary"
            android:fitsSystemWindows="true"
            app:layout_scrollFlags="scroll|enterAlwaysCollapsed">

            <ImageView
                android:id="@+id/fruit_image_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:fitsSystemWindows="true"
                app:layout_collapseMode="parallax" />

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="parallax" />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    ...

</android.support.design.widget.CoordinatorLayout>

```


 2. 在Activity中设置标题和图片

``` java
public class FruitActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fruit);
        ...
        CollapsingToolbarLayout collapsingToolbar = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
        ImageView fruitImageView = (ImageView) findViewById(R.id.fruit_image_view);
        collapsingToolbar.setTitle(fruitName);
        Glide.with(this).load(fruitImageId).into(fruitImageView);
 
    }
}
```



#### TextInputLayout

 1. TextInputLayout输入的时候会自动将隐藏栏推到左上角，用法很简单，只要嵌套一个EditText就可以了

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/et_username"
        android:layout_centerInParent="true">

        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="用户名"
            android:inputType="textPersonName" />

    </android.support.design.widget.TextInputLayout>

</LinearLayout>
```

 2. 添加错误提示在下方

``` java
public class LoginActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.login);

        TextInputLayout usename = (TextInputLayout) findViewById(R.id.et_username);
        usename.setErrorEnabled(true);
        usename.setError("账号错啦");

    }
}
```


#### TabLayout

 1. 开发用的比较多的是TabLayout和ViewPager的联动效果，自己去实现简直麻烦，先定义布局

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.design.widget.TabLayout
        android:id="@+id/sliding_tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabMode="fixed"
        app:tabGravity="fill" />

    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">
    </android.support.v4.view.ViewPager>

</LinearLayout>
```


 2. Activity里面设置属性

``` java
public class LoginActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.login);

        ViewPager pager = (ViewPager)findViewById(R.id.viewPager);
        pager.setAdapter(new MyPagerAdapter(getSupportFragmentManager(),this));

        TabLayout tabLayout = (TabLayout) findViewById(R.id.sliding_tabs);
        tabLayout.addTab(tabLayout.newTab().setText("Tab One"));
        tabLayout.addTab(tabLayout.newTab().setText("Tab Two"));
        tabLayout.addTab(tabLayout.newTab().setText("Tab Three"));
        tabLayout.setTabMode(TabLayout.MODE_FIXED );
        //tabLayout.setTabMode(TabLayout.MODE_SCROLLABLE );
        tabLayout.setupWithViewPager(pager);

    }
}
```
