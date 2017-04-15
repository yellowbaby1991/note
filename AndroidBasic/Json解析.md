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

### JSONObject解析JSON

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



 2. 1

