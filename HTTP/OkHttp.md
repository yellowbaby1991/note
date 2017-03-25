1. [简介](#简介)
2. [配置方法](#配置方法)
3. [HttpGet请求](#httpget请求)
4. [HttpPost请求](#httppost请求)
5. [集合Gson解析JSON数据](#集合gson解析json数据)

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


  [1]: https://github.com/square/okhttp
#### 集合Gson解析JSON数据

 1. 服务器待解析的JSON数据

``` json
[{"id":"5","version":"5.5","name":"Clash of Clans"},
 {"id":"6","version":"7.0","name":"Bom Beach"},
 {"id":"7","version":"3.5","name":"Clash Royale"}]
```

 2. 添加GSON依赖
 
``` gradle
dependencies {
 	...
    compile 'com.google.code.gson:gson:2.7'
    ...
}
```
 3. 创建对应的JAVABean对象

``` java
public class App {

    private String id;

    private String name;

    private String version;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

    @Override
    public String toString() {
        return "App{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", version='" + version + '\'' +
                '}';
    }
}
```

 4. 创建Gson对象调用fromJson方法

``` java
    private void HttpGet() {
        Request request = new Request.Builder().url("http://192.168.87.2/get_data.json").build();
        Call call = mOkHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String responseData = response.body().string();
                Log.d(TAG, responseData);
                parseJSONWithGSON(responseData);
            }
        });
    }

    private void parseJSONWithGSON(String jsonData) {
        Gson gson = new Gson();
        List<App> appList = gson.fromJson(jsonData, new TypeToken<List<App>>() {
        }.getType());
        for (App app : appList) {
            Log.d(TAG, app.toString());
        }
    }
```
