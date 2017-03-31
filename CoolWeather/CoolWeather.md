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

``` javascript
enter code here
```

``` json
{
  "HeWeather": [
	{
      "status": "ok",
	  "basic":{}
	  "aqi":{}
	  "now":{}
	  "suggestion":{}
	  "daily_forecast":{}
	}
  ]
}
```

 2. 有些字段不适合做JAVA类的属性名称，需要使用@SerializedName来指定别名，如Basic类

``` java
public class Basic {

    @SerializedName("city")
    public String cityName;

    @SerializedName("id")
    private String weatherId;

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


 3. 1