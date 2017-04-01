### 简介
　　[android-smart-image-view][1]是一个可以取代原生ImageView的开源组件
### 特性

 1. 支持通过URL来加载图片
 2. 支持从电话簿中加载图片
 3. 异步加载图片
 4. 图片被缓存在内存，以便下次快速加载显示；

### 配置方法

 1. 由于年代久远，只支持加入jar包，[jar包下载地址][2]
 2. 添加网络权限即可
 
``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="android.com.smartimageview">

    <uses-permission android:name="android.permission.INTERNET"></uses-permission>
</manifest>	
```



  [1]: https://github.com/loopj/android-smart-image-view
  [2]: http://loopj.com/android-smart-image-view/