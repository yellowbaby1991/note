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

  [1]: https://github.com/yellowbaby1991/coolweather