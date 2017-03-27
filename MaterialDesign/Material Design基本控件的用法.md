#### 概述
　　Material design 是一种达到可视化，交互性，动效以及多屏幕适应的全面设计，Google为了方便开发者推出了兼容5.0前后的Material design支持库，下面是一些常用的控件
  
#### Toolbar
----------

##### 1.1 基本用法
- [ ] ​Toolbar的出现是为了取代原有的ActionBar，所以想使用Toolbar首先需要将原有的ActionBar隐藏，将style改成NoActionBar
    　
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="yellow.com.materialdesign">

    <application
        ...
        android:theme="@style/AppTheme">
        ...
    </application>

</manifest>
```

``` xml
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.NoActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>
```


