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

#### 环境配置

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

4. 配置litepal.xml

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="android.coolweather.com.coolweather">
    
    <application
        android:name="org.litepal.LitePalApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
    </application>

</manifest>

```


  [1]: https://github.com/yellowbaby1991/coolweather