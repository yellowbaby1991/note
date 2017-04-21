### 概述
　[Dagger2][1]是谷歌公司fork了Dagger1后接受开发后的产品，是Dagger1的升级版，现在由谷歌负责维护，总的来说Dagger2是一个依赖注入的框架，作用和Spring类似，只不过Spring用到了反射来处理，但是在Android中反射会影响性能，Dagger2在编译的时候就已经做好了处理
 
 ### 配置方法

> build.gradle

``` gradle
buildscript {

    ....

    dependencies {

        classpath 'com.android.tools.build:gradle:2.1.0'
        // 添加android-apt 插件
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
```



  [1]: https://github.com/google/dagger