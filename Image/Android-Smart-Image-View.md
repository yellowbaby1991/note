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
 
``` xml?linenums
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="android.com.smartimageview">

    <uses-permission android:name="android.permission.INTERNET"></uses-permission>
</manifest>	
```

### 图片加载

 1. 使用SmartImageView取代ImageView
 

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.loopj.android.image.SmartImageView
        android:id="@+id/my_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</RelativeLayout>
```


 2. 1

  [1]: https://github.com/loopj/android-smart-image-view
  [2]: http://loopj.com/android-smart-image-view/