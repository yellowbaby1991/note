* [将项目托管到GitHub上](#将项目托管到github上)
	* [初次提交](#初次提交)
	* [后续提交](#后续提交)
* [环境配置](#环境配置)
* [遍历全国市县数据](#遍历全国市县数据)
	* [分析](#分析)
	* [细节](#细节)
* [显示天气信息](#显示天气信息)
	* [分析](#分析)
	* [细节](#细节)
* [手动切换天气和城市](#手动切换天气和城市)
	* [分析](#分析)
	* [细节](#细节)
* [后台自动更新天气](#后台自动更新天气)
	* [分析](#分析)
	* [细节](#细节)

### 将项目托管到GitHub上
#### 初次提交
 1. 在GitHub上创建名称为coolweather的 [Repository][1]
 2. 在本地创建一个项目名为CoolWeather的Project
 3. 在本地的CoolWeather里面打开Git Bush，输入下面代码把远程版本克隆到本地

``` git
git clone https://github.com/yellowbaby1991/coolweather.git
```

 4. 进入本地目录将coolweather子文件夹中的.git隐藏文件和readme拷贝到上一层目录，然后删除coolweather子文件夹
 5. 将现有的项目提交到GitHub上
``` git
git add .
git commit -m "First commit."
git push origin master
```

#### 后续提交


 - 命令如下

``` git
git add .
git commit -m "commit"
git push origin master
```


### 环境配置

 1. 加载依赖库，litepal操作数据库，OKhttp网络，GSON解析JSON数据，Glide加载图片
 
``` xml
dependencies {
    ...
    compile 'com.github.bumptech.glide:glide:3.7.0'
    compile 'org.litepal.android:core:1.3.2'
    compile 'com.squareup.okhttp3:okhttp:3.4.1'
    compile 'com.google.code.gson:gson:2.7'
}
```

 2. 数据库设计，三张表，对应三个实体类Province，City，County

``` java
public class Province extends DataSupport {

    private int id;

    private String provinceName;

    private int provinceCode;
	
	//getter and setter	
}	
```


``` java
public class City extends DataSupport {

    private int id;

    private String cityName;

    private int cityCode;

    private int provinceId;
	
	//getter and setter
}	
```

``` java
public class County extends DataSupport {

    private int id;

    private String countyName;

    private String weatherId;

    private int cityId;
	
	//getter and setter
}	
```

 3. 配置litepal.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>
    <dbname value="cool_weather" />
    <version value="1" />
    <list>
        <mapping class="com.coolweather.android.db.Province" />
        <mapping class="com.coolweather.android.db.City" />
        <mapping class="com.coolweather.android.db.County" />
    </list>
</litepal>
```

4. 修改AndroidManifest.xml配置LitePalApplication

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="android.coolweather.com.coolweather">
    
    <application
        android:name="org.litepal.LitePalApplication"
        ...>
    </application>
    ...
</manifest>

```


  [1]: https://github.com/yellowbaby1991/coolweather
### 遍历全国市县数据
#### 分析

 1. 第一次读取会从服务器读取解析Json数据存入本地数据库，随后再读取就不需要再访问服务器，直接读取数据库，从而减少流量消耗
 2. 省，市，县都公用的一个布局，根据当前级别来分布读取数据列表（省，市，县），为了方便显示数据，使用简单的ListView
#### 细节

 1. 布局是简单的垂直线性布局，包含头部文字+返回按钮+ListView列表

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#fff"
    android:fitsSystemWindows="true">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary">

        <TextView
            android:id="@+id/title_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:textColor="#fff"
            android:textSize="20sp"/>

        <Button
            android:id="@+id/back_button"
            android:layout_width="25dp"
            android:layout_height="25dp"
            android:layout_marginLeft="10dp"
            android:layout_alignParentLeft="true"
            android:layout_centerVertical="true"
            android:background="@drawable/ic_back"/>
    </RelativeLayout>

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

 2. 为了使当前布局可以共用，使用一个全局变量记录当前的层，根据选择的层来分布加载数据
 
``` java
public class ChooseAreaFragment extends Fragment {

    /**
     * 当前选中的级别
     */
    private int currentLevel;
    public static final int LEVEL_PROVINCE = 0;
    public static final int LEVEL_CITY = 1;
    public static final int LEVEL_COUNTY = 2;
	
	@Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                if (currentLevel == LEVEL_PROVINCE) {
					//当前级为省，点击后就查询市
                    selectedProvince = provinceList.get(position);
                    queryCities();
                } else if (currentLevel == LEVEL_CITY) {
				   //当前级为市，点击后就查询省
                    selectedCity = cityList.get(position);
                    queryCounties();
                }
            }
        });
		...
        queryProvinces();//第一次加载查询省
    }

}
```


 3. 查询方法的逻辑，首先读取数据库，如果有数据，就更新dataList，如果没有就去服务器加载到数据库后再查询一次

``` java
public class ChooseAreaFragment extends Fragment {

    /**
     * ListView需要的数据源
     */
    private List<String> dataList = new ArrayList<>();
	
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.choose_area, container, false);
		....
        listView = (ListView) view.findViewById(R.id.list_view);
        adapter = new ArrayAdapter<String>(getContext(), android.R.layout.simple_list_item_1, dataList);
        listView.setAdapter(adapter);
        return view;
    }

    private void queryProvinces() {
        titleText.setText("中国");
        backButton.setVisibility(View.GONE);
		//从数据库查询
        provinceList = DataSupport.findAll(Province.class);
        if (provinceList.size() > 0) {//如果本地数据库有数据
            dataList.clear();
            for (Province province : provinceList) {
                dataList.add(province.getProvinceName());
            }
            adapter.notifyDataSetChanged();//更新listview
            listView.setSelection(0);
            currentLevel = LEVEL_PROVINCE;
        } else {//否则去服务器查询
            String address = "http://guolin.tech/api/china";
            queryFromServer(address, "province");
        }
    }
}
```


 4. 从服务端加载的逻辑，调用封装好的OkHttp工具类，将得到的JSON数据解析为对象，利用LitePal存入本地数据库，最后切回主线程刷新

``` java
public class ChooseAreaFragment extends Fragment {

    private void queryFromServer(String address, final String type) {
        showProgressDialog();
        HttpUtil.sendOkHttpRequest(address, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
               ...//加载失败
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String responseText = response.body().string();
                boolean result = false;
                if ("province".equals(type)) {
                    result = Utility.handleProvinceResponse(responseText);
                } else if ("city".equals(type)) {
                    result = Utility.handleCityResponse(responseText, selectedProvince.getId());
                } else if ("county".equals(type)) {
                    result = Utility.handleCountyResponse(responseText, selectedCity.getId());
                }

                if (result) {
                  ...//如果下载成功就切回主线程刷新查询一次
                }

            }
        });
    }
}
```

``` java
public class Utility {
    /**
     * 解析和处理服务器返回的省级数据
     */
    public static boolean handleProvinceResponse(String response) {
        if (!TextUtils.isEmpty(response)) {
            try {
                JSONArray allProvinces = new JSONArray(response);
                for (int i = 0; i < allProvinces.length(); i++) {
                    JSONObject provinceObject = allProvinces.getJSONObject(i);
                    Province province = new Province();
                    province.setProvinceName(provinceObject.getString("name"));
                    province.setProvinceCode(provinceObject.getInt("id"));
                    province.save();//使用LitePal存入本地数据库
                }
                return true;
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
        return false;
    }
	....
}
```
### 显示天气信息
#### 分析

 1. 从根据weather_id从服务器下载对应的json文件解析成实体类Weather展示在布局中
 2. 加载的天气解析文件存在SharedPreferences中缓存，下次加载时优先查询本地，本地没有再从网上拉取
 3. 背景图的更换采取拉取必应的每日一图，拉取地址也存在本地缓存，下次加载时候使用Glide直接加载

#### 细节

 1. 带解析的json文件格式如下，所以需要对应5个实体类+一个Weather类

``` java
public class Weather {

    public String status;

    public Basic basic;

    public AQI aqi;

    public Now now;

    public Suggestion suggestion;

    @SerializedName("daily_forecast")
    public List<Forecast> forecastList;
	
}
```

``` json
{
  "HeWeather": [
	{
      "status": "ok",
	  "basic": {}
	  "aqi": {}
	  "now": {}
	  "suggestion": {}
	  "daily_forecast": {}
	}
  ]
}

```

``` javascript
public class Utility {

    /**
     * 将返回的JSON数据解析成Weather实体类
     */
    public static Weather handlerWeatherResponse(String response) {
        try {
            JSONObject jsonObject = new JSONObject(response);
            JSONArray jsonArray = jsonObject.getJSONArray("HeWeather");
            String weatherContent = jsonArray.getJSONObject(0).toString();
            return new Gson().fromJson(weatherContent, Weather.class);
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return null;
    }
	...
}
```


 2. 有些字段不适合做JAVA类的属性名称，需要使用@SerializedName来指定别名，如Basic类

``` java
public class Basic {

    @SerializedName("city")
    public String cityName;

    @SerializedName("id")
    public String weatherId;

    public Update update;

    public class Update {
        @SerializedName("loc")
        public String updateTime;
    }
	
}
```

``` json
"basic": {
        "city": "苏州",
        "id": "CN101190401",	  
        "update": {
          "loc": "2017-03-31 09:52",	
		 }
}
```

 3. 天气布局文件比较长，所以分解成了四个子布局

``` xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/colorPrimary">

    <android.support.v4.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/swipe_refresh"
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <ScrollView
                android:id="@+id/weather_layout"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scrollbars="none"
                android:overScrollMode="never">

                <LinearLayout
                    android:orientation="vertical"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:fitsSystemWindows="true">

                    <include layout="@layout/title" />

                    <include layout="@layout/now" />

                    <include layout="@layout/forecast" />

                    <include layout="@layout/aqi" />

                    <include layout="@layout/suggestion" />

                </LinearLayout>

            </ScrollView>

        </android.support.v4.widget.SwipeRefreshLayout>
		
        <!--侧滑菜单-->
        <fragment
            android:id="@+id/choose_area_fragment"
            android:name="com.coolweather.android.fragment.ChooseAreaFragment"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            />

    </android.support.v4.widget.DrawerLayout>

</FrameLayout>

```

 4. 判断本地是否有缓存来加载数据到界面

``` java
public class WeatherActivity extends AppCompatActivity {

    ...//控件声明

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_weather);
        ...//控件初始化

        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
        String weatherString = prefs.getString("weather", null);

        if (weatherString != null) {
           //有缓存时直接解析天气数据
            Weather weather = Utility.handlerWeatherResponse(weatherString);
            showWeatherInfo(weather);
        } else {
           //无缓存时服务器拉取
            String weatherId = getIntent().getStringExtra("weather_id");
            weatherLayout.setVisibility(View.INVISIBLE);
            requestWeather(weatherId);
        }
    }


    /**
     * 根据天气ID请求城市天气信息
     */
    private void requestWeather(String weatherId) {
        String weatherUrl = "http://guolin.tech/api/weather?cityid=" + weatherId + "&key=567b365d3bdf4ca295d2a46745c1a817";
        HttpUtil.sendOkHttpRequest(weatherUrl, new Callback() {
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                final String responseText = response.body().string();
                final Weather weather = Utility.handlerWeatherResponse(responseText);
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        //拉取成功后把数据存入本地
                        if (weather != null && "ok".equals(weather.status)) {
                            SharedPreferences.Editor editor = PreferenceManager.getDefaultSharedPreferences(WeatherActivity.this).edit();
                            editor.putString("weather", responseText);
                            editor.apply();
                            showWeatherInfo(weather);
                        } else {
                            Toast.makeText(WeatherActivity.this, "获取天气信息失败", Toast.LENGTH_SHORT).show();
                        }
                    }
                });
            }

            @Override
            public void onFailure(Call call, IOException e) {
               //...拉取失败
            }
        });
        loadBingPic();
    }

    private void showWeatherInfo(Weather weather) {
        String cityName = weather.basic.cityName;
        String updateTime = weather.basic.update.updateTime.split(" ")[1];
        String degree = weather.now.temperature + "℃";
        String weatherInfo = weather.now.more.info;
        titleCity.setText(cityName);
        titleUpdateTime.setText(updateTime);
        degreeText.setText(degree);
        weatherInfoText.setText(weatherInfo);
        forecastLayout.removeAllViews();
        for (Forecast forecast : weather.forecastList) {
            View view = LayoutInflater.from(this).inflate(R.layout.forecast_item, forecastLayout, false);
            TextView dateText = (TextView) view.findViewById(R.id.date_text);
            TextView infoText = (TextView) view.findViewById(R.id.info_text);
            TextView maxText = (TextView) view.findViewById(R.id.max_text);
            TextView minText = (TextView) view.findViewById(R.id.min_text);
            dateText.setText(forecast.date);
            infoText.setText(forecast.more.info);
            maxText.setText(forecast.temperature.max);
            minText.setText(forecast.temperature.min);
            forecastLayout.addView(view);
        }
        if (weather.aqi != null) {
            aqiText.setText(weather.aqi.city.aqi);
            pm25Text.setText(weather.aqi.city.pm25);
        }
        String comfort = "舒适度：" + weather.suggestion.comfort.info;
        String carWash = "洗车指数：" + weather.suggestion.carWash.info;
        String sport = "运行建议：" + weather.suggestion.sport.info;
        comfortText.setText(comfort);
        carWashText.setText(carWash);
        sportText.setText(sport);
        weatherLayout.setVisibility(View.VISIBLE);
    }
}
```

 5. 每日一图，从必应拉取图片显示为背景图

``` xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/colorPrimary">

    <!--使用的是一个ImageView和FrameLayout配合实现背景-->
    <ImageView
        android:id="@+id/bing_pic_img"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="centerCrop" />

    <android.support.v4.widget.DrawerLayout
    <!--天气界面-->

    <!--侧滑菜单-->
    </android.support.v4.widget.SwipeRefreshLayout>


</FrameLayout>

```

``` java
package com.coolweather.android;

import android.content.SharedPreferences;
import android.coolweather.com.coolweather.R;
import android.graphics.Color;
import android.os.Build;
import android.os.Bundle;
import android.preference.PreferenceManager;
import android.support.v7.app.AppCompatActivity;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;

import com.bumptech.glide.Glide;
import com.coolweather.android.gson.Forecast;
import com.coolweather.android.gson.Weather;
import com.coolweather.android.until.HttpUtil;
import com.coolweather.android.until.Utility;

import java.io.IOException;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.Response;

public class WeatherActivity extends AppCompatActivity {

    ...
    private ImageView bingPicImg;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ...

        setContentView(R.layout.activity_weather);

        ...
        bingPicImg = (ImageView) findViewById(R.id.bing_pic_img);

        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
        String bingPic = prefs.getString("bing_pic", null);
        if (bingPic != null) {
            Glide.with(this).load(bingPic).into(bingPicImg);
        } else {
            loadBingPic();
        }
        ...
    }

    /**
     * 加载必应每日一图
     */
    private void loadBingPic() {
        String requestBingPic = "http://guolin.tech/api/bing_pic";
        HttpUtil.sendOkHttpRequest(requestBingPic, new Callback() {
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                //从服务器得到每日一图的URL，然后切回主线程使用Glide加载
                final String bingPic = response.body().string();
                SharedPreferences.Editor editor = PreferenceManager.getDefaultSharedPreferences(WeatherActivity.this).edit();
                editor.putString("bing_pic", bingPic);
                editor.apply();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Glide.with(WeatherActivity.this).load(bingPic).into(bingPicImg);
                    }
                });
            }

            @Override
            public void onFailure(Call call, IOException e) {
                e.printStackTrace();
            }
        });
    }

   ...
}

```

6. 去掉状态栏，是背景图全屏显示

``` java
public class WeatherActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        //取消标题
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        //取消状态栏
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
        WindowManager.LayoutParams.FLAG_FULLSCREEN);

        setContentView(R.layout.activity_weather);
		
		}
		...
}
```


### 手动切换天气和城市
#### 分析

 1. 使用DrawerLayout来实现侧滑选择城市，并且加入顶部导航按钮
 2. 使用SwipeRefreshLayout来实现下拉刷新效果

#### 细节

 1.到现在为止，整个天气界面的布局如下，整体为FrameLayout，ImageView作为背景图，然后一个DrawerLayout侧滑菜单包含了一个刷新下拉列表SwipeRefreshLayout

``` xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/colorPrimary">

    <!--背景图-->
    <ImageView
        android:id="@+id/bing_pic_img"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="centerCrop" />

    <!--整个侧滑布局-->
    <android.support.v4.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!--天气详情-->
        <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/swipe_refresh"
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <ScrollView
                android:id="@+id/weather_layout"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scrollbars="none"
                android:overScrollMode="never">

                <LinearLayout
                    android:orientation="vertical"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:fitsSystemWindows="true">

                    <include layout="@layout/title" />

                    <include layout="@layout/now" />

                    <include layout="@layout/forecast" />

                    <include layout="@layout/aqi" />

                    <include layout="@layout/suggestion" />

                </LinearLayout>

            </ScrollView>

        </android.support.v4.widget.SwipeRefreshLayout>

        <!--侧滑菜单-->
        <fragment
            android:id="@+id/choose_area_fragment"
            android:name="com.coolweather.android.fragment.ChooseAreaFragment"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            />

    </android.support.v4.widget.DrawerLayout>

</FrameLayout>


```


 2.添加顶部菜单按钮提醒用户有个侧滑菜单
 

``` java
public class WeatherActivity extends AppCompatActivity {

    ...
    private Button navButton;
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
		...
        navButton = (Button) findViewById(R.id.nav_button);		
        navButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {//点击后显示菜单
                drawerLayout.openDrawer(GravityCompat.START);
            }
        });		
```


 3.点击县列表的时候根据当前Activity的类型来判断是第一次打开还是重复选择刷新

``` java
public class ChooseAreaFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
			...
			if (currentLevel == LEVEL_COUNTY) {//点击的是县
				String weatherId = countyList.get(position).getWeatherId();
				if (getActivity() instanceof MainActivity) {//第一次打开天气界面
					Intent intent = new Intent(getActivity(), WeatherActivity.class);
					intent.putExtra("weather_id", weatherId);
					startActivity(intent);
					getActivity().finish();
				} else if (getActivity() instanceof WeatherActivity) {//再次选择某个县的天气
					WeatherActivity activity = (WeatherActivity) getActivity();
					activity.drawerLayout.closeDrawers();
					activity.swipeRefresh.setRefreshing(true);
					activity.requestWeather(weatherId);
				}
                }
            }
        });
        ...
        });
    }

}
```


### 后台自动更新天气
#### 分析

 1. 创建一个长期在后台运行的定时服务来实现每8小时来更新一次
 2. 刷新天气数据：根据本地存储的weather json数据解析得到的weatherId再去网上拉取新的json数据存在本地即可
 3. 刷新图片：从服务器拉取最新的图片URL存入本地

#### 细节

 1. 创建定时服务类，每8小时更新一次天气和图片
 
``` java
public class AutoUpdateService extends Service {

    public AutoUpdateService() {
    }

    //定时循环服务
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        updateWeather();
        updateBingPic();
        AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
        int anHour = 8 * 60 * 60 * 1000;
        long triggerAtTime = SystemClock.elapsedRealtime() + anHour;
        Intent i = new Intent(this, AutoUpdateService.class);
        PendingIntent pi = PendingIntent.getService(this, 0, i, 0);
        manager.cancel(pi);
        manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pi);
        return super.onStartCommand(intent, flags, startId);
    }

    /**
     * 更新必应每日一图
     */
    private void updateBingPic() {
        String requestBingPic = "http://guolin.tech/api/bing_pic";
        HttpUtil.sendOkHttpRequest(requestBingPic, new Callback() {
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                final String bingPic = response.body().string();
                SharedPreferences.Editor editor = PreferenceManager.getDefaultSharedPreferences(AutoUpdateService.this).edit();
                editor.putString("bing_pic", bingPic);
                editor.apply();
            }

            @Override
            public void onFailure(Call call, IOException e) {
                e.printStackTrace();
            }
        });
    }

    /**
     * 更新天气信息
     */
    private void updateWeather() {
        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
        String weatherString = prefs.getString("weather", null);
        if (weatherString != null) {
            Weather weather = Utility.handlerWeatherResponse(weatherString);
            String weatherId = weather.basic.weatherId;
            String weatherUrl = "http://guolin.tech/api/weather?cityid=" + weatherId + "&key=567b365d3bdf4ca295d2a46745c1a817";
            HttpUtil.sendOkHttpRequest(weatherUrl, new Callback() {
                @Override
                public void onResponse(Call call, Response response) throws IOException {
                    String responseText = response.body().string();
                    Weather weather = Utility.handlerWeatherResponse(responseText);
                    if (weather != null && "ok".equals(weather.status)) {
                        SharedPreferences.Editor editor = PreferenceManager.getDefaultSharedPreferences(AutoUpdateService.this).edit();
                        editor.putString("weather", responseText);
                        editor.apply();
                    }
                }

                @Override
                public void onFailure(Call call, IOException e) {
                    e.printStackTrace();
                }
            });
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

 2. 在Activity中开启服务
 
``` java
		//开启定时更新服务
		Intent intent = new Intent(this, AutoUpdateService.class);
		startService(intent);
```

