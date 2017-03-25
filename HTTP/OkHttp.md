#### 简介
　　OkHttp是一个处理网络请求的开源项目,是安卓端最火热的轻量级框架,由移动支付Square公司贡献
　　[GitHub地址][1]


#### 配置方法

app\build.gradle中添加依赖

``` xml
dependencies {
    ...
    compile 'com.squareup.okhttp3:okhttp:3.4.1'
    ...
}
```

#### HttpGet请求

 1. 异步请求

``` java
OkHttpClient mOkHttpClient = new OkHttpClient();
Request request = new Request.Builder().url("https://github.com/square/okhttp").build();
Call call = mOkHttpClient.newCall(request);
//异步请求
call.enqueue(new Callback() {
	@Override
	public void onFailure(Request request, IOException e) {
	}

	@Override
	public void onResponse(Response response) throws IOException {
		String htmlStr = response.body().string();
		Log.i(TAG, "onResponse: " + htmlStr);
	}
});
```

 2. 同步请求

``` java
OkHttpClient mOkHttpClient = new OkHttpClient();
Request request = new Request.Builder().url("https://github.com/square/okhttp").build();
Call call = mOkHttpClient.newCall(request);
Response response = call.execute();
String htmlStr = response.body().string();
Log.i(TAG, "onResponse: " + htmlStr);
```

























  [1]: https://github.com/square/okhttp
  
#### HttpPost请求
　　和get相比多了Request多了一个RequestBody
``` java
RequestBody requestBody = new FormBody.Builder()
                .add("username", "admin")
                .add("password", "123456")
                .build();
Request request = new Request.Builder().url("https://github.com/square/okhttp").post(requestBody).build();
//.... 同步or异步
```
