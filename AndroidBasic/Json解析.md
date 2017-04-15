* [什么是JSON](#什么是json)
* [JSON的格式](#json的格式)
* [使用JSONObject解析JSON](#使用jsonobject解析json)
* [使用Gson解析JSON](#使用gson解析json)

### 什么是JSON
　JSON(JavaScript Object Notation) 是一种轻量量级的数据交换格式。 其采用完全独立于语言的文本格式，但是也使用了了类似于C语言家族的习惯。这些特性使JSON成为理理想的数据交换语言。 易易于人阅读和编写，同时也易易于机器器解析和生成(一般用于提升网络传输速率)
 
 ### JSON的格式
 　　在AS中创建json文件方便格式化查看，存在main/assets
   
 1. 单个对象
 
``` json
{"firstName":"Brett","lastName":"McLaughlin","email":"aaaa"}
```

 2. 简单队列

``` json
[
  {
    "firstName": "Brett",
    "lastName": "McLaughlin",
    "email": "aaaa"
  },
  {
    "firstName": "Jason",
    "lastName": "Hunter",
    "email": "bbbb"
  },
  {
    "firstName": "Elliotte",
    "lastName": "Harold",
    "email": "cccc"
  }
]
```

### 使用JSONObject解析JSON

 1. 解析单个对象
 
``` json
{
  "userbean": {
    "name": "yellow",
    "sex": "男",
    "age": "25"
  }
}
```


``` java
JSONObject jsonObject = new JSONObject(jsonObjectString).getJSONObject("userbean");
String name = jsonObject.getString("name");
String sex = jsonObject.getString("sex");
String age = jsonObject.getString("age");
```


 2. 解析队列
 
``` json
{
  "userlist": [
    {
      "name": "yellow",
      "sex": "男",
      "age": "25"
    },
    {
      "name": "baby",
      "sex": "男",
      "age": "25"
    }
  ]
}
```

``` java
JSONArray jsonArray = new JSONObject(jsonObjectString).getJSONArray("userlist");
List<User> userList = new ArrayList<>();
for (int i = 0; i < jsonArray.length(); i++) {
	JSONObject jsonObject = (JSONObject) jsonArray.opt(i);
	String name = jsonObject.getString("name");
	String sex = jsonObject.getString("sex");
	String age = jsonObject.getString("age");
	User user = new User(name, sex, age);
	userList.add(user);
}
```



### 使用Gson解析JSON
　Gson是Google提供的一个针对json的开发包，方便Bean和JSON之间的互转，使用需要添加依赖
 
``` xml
dependencies {
 	...
    compile 'com.google.code.gson:gson:2.7'
}
```
#### Gson基本用法
 1. 解析单个对象

``` json
{
  "name": "yellow",
  "sex": "男",
  "age": "25"
}
```

``` java
Gson gson = new Gson();
UserBean userBean = gson.fromJson(jsonObjectString,UserBean.class);
```


 2. 解析队列
 
``` json
[
  {
    "name": "yellow",
    "sex": "男",
    "age": "25"
  },
  {
    "name": "baby",
    "sex": "男",
    "age": "25"
  }
]
```

``` java
Gson gson = new Gson();
List<UserBean> userBeans = gson.fromJson(jsonObjectString, new TypeToken<List<UserBean>>() {
}.getType());
Toast.makeText(this, userBeans.toString(), Toast.LENGTH_SHORT).show();
```


 3. 对象/数组转JSON
 
``` java
public static String obj2Json(Object obj) {
    Gson gson = new Gson();
    return gson.toJson(obj);
}
```

#### 使用GsonFormat快速生成javabean代码

 1. 安装：setting-plugin-GsonFormat
 2. 使用：创建一个空的JAVA类，然后generate，把json拷贝进去即可
 　
 
 
### 使用FastJson解析JSON
　[FastJson][1]是阿里巴巴的一个JSON处理包，号称速度最快，无依赖
 
 1. 配置方法
 
``` xml
compile 'com.alibaba:fastjson:1.1.56.android'
```


 2. 1

  [1]: https://github.com/alibaba/fastjson