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

 2. 打印变量

  [1]: https://github.com/orhanobut/logger