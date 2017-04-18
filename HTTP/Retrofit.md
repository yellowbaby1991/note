* [概述](#概述)
* [配置方法](#配置方法)
* [使用方法](#使用方法)
* [Retrofit+GsonConverterFactory](#retrofitgsonconverterfactory)

### 概述
　[Retrofit][1]　是现在最流行的网络请求框架，同样出自Square公司，对okhttp做了一层封装。把网络请求都交给给了Okhttp，我们只需要通过简单的配置就能使用retrofit来进行网络请求了，其主要作者是Android大神JakeWharton

### 配置方法

``` gradle
dependencies {
    ...
    compile 'com.squareup.retrofit2:retrofit:2.2.0'
}
```

### 使用方法

> 基本使用步骤

 1. 创建Retrofit对象
 2. 创建代理类
 2. 创建访问请求
 3. 发送请求
 4. 处理结果

> Get请求接口 
 
``` java
public interface GitHubApi {
    // https://api.github.com/repos/square/retrofit/contributors
    @GET("repos/{owner}/{repo}/contributors")
    Call<ResponseBody> contributorsBySimpleGetCall(@Path("owner") String owner, @Path("repo") String repo);
}

```

> Post请求接口 
 
``` java
public interface GitHubApi {
    // https://api.github.com/repos/square/retrofit/contributors
    @POST("repos/{owner}/{repo}/contributors")
    Call<ResponseBody> contributorsBySimpleGetCall(@Path("owner") String owner, @Path("repo") String repo);
}

```

> 发送请求

``` java
//1. 创建Retrofit对象
Retrofit retrofit = new Retrofit.Builder().baseUrl("https://api.github.com/").build();

//2. 创建代理类
GitHubApi repo = retrofit.create(GitHubApi.class);

//3. 创建访问请求
Call<ResponseBody> call = repo.contributorsBySimpleGetCall("square","retrofit");

//4. 发送请求
call.enqueue(new Callback<ResponseBody>() {
	@Override
	public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
	   //5. 处理结果
	}

	@Override
	public void onFailure(Call<ResponseBody> call, Throwable t) {

	}
});
```

### Retrofit+GsonConverterFactory
　使用GsonConverterFactory可以直接将返回的json转换成对象
> app.build

``` gradle
dependencies {
    compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
    compile 'com.squareup.retrofit2:retrofit:2.2.0'
    compile 'com.jakewharton:butterknife:8.5.1'
    annotationProcessor 'com.jakewharton:butterknife-compiler:8.5.1'
}
```

> activity_main.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin">

    <Button
        android:id="@+id/click_me_bt"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:padding="5dp"
        android:text="点我"
        android:textSize="16sp"/>
    <TextView
        android:id="@+id/result_tv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@id/click_me_bt"
        android:text="Hello World!"
        android:textSize="16sp"/>
</RelativeLayout>
```

> MovieEntity.java

``` java
public class MovieEntity {

    private int count;
    private int start;
    private int total;
    private String title;

    @Override
    public String toString() {
        return "MovieEntity{" +
                "count=" + count +
                ", start=" + start +
                ", total=" + total +
                ", title='" + title + '\'' +
                '}';
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public int getStart() {
        return start;
    }

    public void setStart(int start) {
        this.start = start;
    }

    public int getTotal() {
        return total;
    }

    public void setTotal(int total) {
        this.total = total;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

> MovieService.java

``` java
public interface MovieService {
    @GET("top250")
    Call<MovieEntity> getTopMovie(@Query("start") int start,@Query("count") int count);
}
```

> MainActivity.java

``` java
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.click_me_bt)
    Button mClickMe;
    @BindView(R.id.result_tv)
    TextView mResultTv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }

    @OnClick(R.id.click_me_bt)
    public void onClick() {
        getMovie();
    }

    //进行网络请求
    private void getMovie() {
        String baseUrl = "https://api.douban.com/v2/movie/";
        Retrofit retrofit = new Retrofit.Builder().baseUrl(baseUrl).addConverterFactory(GsonConverterFactory.create()).build();
        MovieService movieService = retrofit.create(MovieService.class);
        Call<MovieEntity> call = movieService.getTopMovie(0, 10);
        call.enqueue(new Callback<MovieEntity>() {
            @Override
            public void onResponse(Call<MovieEntity> call, Response<MovieEntity> response) {
                mResultTv.setText(response.body().toString());
            }

            @Override
            public void onFailure(Call<MovieEntity> call, Throwable t) {
                t.printStackTrace();
                mResultTv.setText(t.getMessage());
            }
        });

    }

```


  [1]: https://github.com/square/retrofit