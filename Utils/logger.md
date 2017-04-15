### 概述
　[logger][1] 是一款著名的开源日志库框架，和原生比，具备打印以下信息的功能
 
 1. 线程的信息
 2. 类的信息
 3. 方法的信息
 4. 格式打印json、xml等
 5. 点击链接跳转到源码打印处

### 配置方法

``` java
dependencies {
  compile 'com.orhanobut:logger:1.15'
}
```

### 基本用法

 1. 打印变量

``` java
String userName = "Jerry";
Logger.i(userName);
```

 2. 设置标签
 ``` java
Logger.init("MainActivity");
String userName = "Jerry";
Logger.i(userName);
```
 3. 给当前打印的换一个单独的tag名
 
``` java
Logger.init("MainActivity");
String userName = "Jerry";
Logger.i(userName);
// 给当前打印的换一个单独的tag名
Logger.t("App").i(userName);
Logger.e(userName);
```

 4. 打印多种类型数据

``` java
Logger.d("hello");
Logger.e("hello");
Logger.w("hello");
Logger.v("hello");
Logger.wtf("hello");
// 打印json格式
String json = createJson().toString();
Logger.json(json);
// 打印xml格式
Logger.xml(XML_CONTENT);
// 打印自定义级别、tag、信息等格式日志
Logger.log(DEBUG, "tag", "message", throwable);
```
 
 5. 打印数组、List、map等对象数据
 
``` java
String[] names = {"Jerry", "Emily", "小五", "hongyang", "七猫"};
Logger.d(names);  // 打印字符数组
List<User> users = new ArrayList<>();
for (int i = 0; i < names.length; i++) {    
    User user = new User(names[i], 10 + i);
    users.add(user);
}
Logger.d(users);  // 打印List
```


  [1]: https://github.com/orhanobut/logger