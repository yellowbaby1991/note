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

//一个漂亮的三方log库
compile 'com.orhanobut:logger:1.15'

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

    public void showCitys(List<CityData> cityDatas);
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
    public void showCitys(List<CityData> cityDatas) {

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
        List<CityData> cityDatas = new ArrayList();

        //主线程
        getView().showCitys(cityDatas);

        //主线程
        getView().hideProgress();
    }
}
```


 5. 至此为止，基本框架搭建完毕，MVP的设计思路：Activity实现V接口，调用P层逻辑，P层从M层拿到数据后分发给V层

## 使用RxJava+Retrofit来请求数据

 1. 创建RetrofitUtil
 
``` java
public class RetrofitUtil {

    private Retrofit mRetrofit;
    private static RetrofitUtil mInstance;

    private RetrofitUtil() {
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        mRetrofit = new Retrofit.Builder()
                .client(builder.build())
                .baseUrl("http://guolin.tech")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();
    }

    public static RetrofitUtil getInstance() {
        if (mInstance == null) {
            synchronized (RetrofitUtil.class) {
                mInstance = new RetrofitUtil();
            }
        }
        return mInstance;
    }

    public static CityService getCityService() {
        return getInstance().mRetrofit.create(CityService.class);
    }
}
```


 2. 创建CityService
 
``` java
public interface CityService {
    @GET("http://guolin.tech/api/china")
    Observable<List<CityData>> getCityData();
}
```

 3. 暂时在P层进行网络请求

``` java
public class CityPresenter extends MvpBasePresenter<CityView> {

    public void loadCity() {

        //主线程
        getView().showProgress();

        RetrofitUtil
                .getCityService()
                .getCityData()//IO线程得到数据
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<List<CityData>>() {//切到主线程
                    @Override
                    public void accept(List<CityData> cityDatas) throws Exception {
                        Logger.d(cityDatas);
                        getView().showCity(cityDatas);
                        getView().hideProgress();
                    }
                });


    }
}
```

 4. 将P层数据获取代码抽到M层
 
``` java
public class CityModel {

    public Observable<List<CityData>> getCityDatas() {
        return RetrofitUtil
                .getCityService()
                .getCityData()//IO线程得到数据
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread());
    }

}
```

 5. P层代码精简为如下
 
``` java
public class CityPresenter extends MvpBasePresenter<CityView> {

    private CityModel mCityModel;

    public CityPresenter() {
        mCityModel = new CityModel();
    }

    public void loadCity() {

        //主线程
        getView().showProgress();

        mCityModel.getCityDatas()
                .subscribe(new Consumer<List<CityData>>() {//切到主线程
                    @Override
                    public void accept(List<CityData> cityDatas) throws Exception {
                        Logger.d(cityDatas);
                        getView().showCity(cityDatas);
                        getView().hideProgress();
                    }
                });


    }
}
```


  [1]: http://gank.io/post/560e15be2dca930e00da1083
  [2]: https://github.com/square/retrofit
