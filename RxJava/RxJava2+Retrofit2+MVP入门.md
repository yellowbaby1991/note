# RxJava2+Retrofit2+MVP
## 概述

> 概念介绍

 - RxJava：当下最火的响应式编程库，[入门指导][1]
 - Retrofit：Square出品封装在okhttp之上的新一代最火网络请求库，[Github地址][2]
 - MVP：MVC升级版，最适合Android项目的架构

> 本文目的：

实战一个完整的全国城市查询demo来实现RxJava2+Retrofit2+MVP架构


## 依赖

``` java
//rxjava2
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'

//retrofit2
compile 'com.squareup.retrofit2:retrofit:2.2.0'
compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
compile 'com.squareup.retrofit2:converter-gson:2.2.0'

//著名的butterknife
compile 'com.jakewharton:butterknife:8.5.1'

//快速实现mvp的三方库
compile 'com.hannesdorfmann.mosby:mvp:2.0.1'
compile 'com.hannesdorfmann.mosby:viewstate:2.0.1'
```

## 网络接口

> API

http://guolin.tech/api/china

> 根据json生成bean类

``` java
public class CityData {

    private int id;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

## 搭建MVP框架

 1. 首先创建View层，只用来显示数据
 
``` java
public interface CityView extends MvpView {

    public void showProgress();

    public void hideProgress();

    public void loadWeather(CityData cityData);
}
```

 2. 然后创建P层，用来负责逻辑跳转
 

``` java
public class CityPresenter extends MvpBasePresenter<CityView> {

    public void loadCity() {

    }
	
}
```

 3. 所以Activity现在的状态应该是这样

``` java
public class MainActivity extends MvpActivity<CityView, CityPresenter> implements CityView {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        presenter.loadCity();

    }

    @Override
    public CityPresenter createPresenter() {
        return new CityPresenter();
    }

    @Override
    public void showProgress() {

    }

    @Override
    public void hideProgress() {

    }

    @Override
    public void loadWeather(CityData cityData) {

    }
}
```

 4. 然后完善P层的逻辑

``` java
public class CityPresenter extends MvpBasePresenter<CityView> {

    public void loadCity() {
        //主线程
        getView().showProgress();

        //I/Ox线程,从网络获取，交给M层去处理
        CityData cityData = new CityData();

        //主线程
        getView().loadWeather(cityData);

        //主线程
        getView().hideProgress();
    }
}
```


 5. 至此为止，MVP全部逻辑如下




返回的json：

  [1]: http://gank.io/post/560e15be2dca930e00da1083
  [2]: https://github.com/square/retrofit
