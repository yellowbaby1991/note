### 简介
　[CircleImageView][1]一个圆形图片的开源控件
　
 ![enter description here][2]

### 配置方法

``` xml
dependencies {  
    ...
    compile 'de.hdodenhof:circleimageview:2.1.0'  
} 
```

### 使用方法

 1. 布局定义
 
``` xml
    <de.hdodenhof.circleimageview.CircleImageView
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/profile_image"
        android:layout_width="96dp"
        android:layout_height="96dp"
        android:src="@drawable/boy"
        app:civ_border_width="2dp"
        app:civ_border_color="#FF000000"/>
```


 2. 属性说明
 

``` xml
<declare-styleable name="CircleImageView">  
        <attr name="civ_border_width" format="dimension" />//设置边框的宽度，默认为0，即无边框
        <attr name="civ_border_color" format="color" />//设置边框的颜色，默认为黑色
        <attr name="civ_border_overlay" format="boolean" />//设置边框是否覆盖在图片上，默认为false，即边框在图片外圈
        <attr name="civ_fill_color" format="color" /> //设置图片的底色，默认透明 
</declare-styleable> 
```


  [1]: https://github.com/hdodenhof/CircleImageView
  [2]: https://camo.githubusercontent.com/e17a2a83e3e205a822d27172cb3736d4f441344d/68747470733a2f2f7261772e6769746875622e636f6d2f68646f64656e686f662f436972636c65496d616765566965772f6d61737465722f73637265656e73686f742e706e67