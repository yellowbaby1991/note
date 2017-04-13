### 简介
　[RoundedImageView][1]是一个速度很快支持各种圆角，椭圆角图片的ImageView
 
　![enter description here][2]

### 配置方法

``` xml
dependencies {
    compile 'com.makeramen:roundedimageview:2.3.0'
}
```

### 使用方法

 1. 在布局中使用
 
``` xml
<com.makeramen.roundedimageview.RoundedImageView xmlns:app="http://schemas.android.com/apk/res-auto"
	android:id="@+id/imageView1"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:scaleType="fitCenter"
	android:src="@mipmap/ic_launcher"
	app:riv_border_color="#333333"
	app:riv_border_width="2dip"
	app:riv_corner_radius="100dip"
	app:riv_mutate_background="true"
	app:riv_oval="false"
	app:riv_tile_mode="repeat" />
```

 2. 参数含义

``` xml
    <!-- 
        corner_radius         边缘的弧度数
        border_width         描边的宽度
        border_color         描边的颜色
        mutate_background     背景是否变化
        oval                 是否为圆形
        android:scaleType     拉伸的方式
     -->
    <declare-styleable name="RoundedImageView">
        <attr name="corner_radius" format="dimension" />
        <attr name="border_width" format="dimension" />
        <attr name="border_color" format="color" />
        <attr name="mutate_background" format="boolean" />
        <attr name="oval" format="boolean" />
        <attr name="android:scaleType" />
    </declare-styleable>
```

 3. 在代码中使用

``` java
RoundedImageView riv = new RoundedImageView(context);
riv.setScaleType(ScaleType.CENTER_CROP);
riv.setCornerRadius((float) 10);
riv.setBorderWidth((float) 2);
riv.setBorderColor(Color.DKGRAY);
riv.mutateBackground(true);
riv.setImageDrawable(drawable);
riv.setBackground(backgroundDrawable);
riv.setOval(true);
riv.setTileModeX(Shader.TileMode.REPEAT);
riv.setTileModeY(Shader.TileMode.REPEAT);
```


  [1]: https://github.com/vinc3m1/RoundedImageView
  [2]: https://camo.githubusercontent.com/ed1e075be6ed97fa9091d3702e9b96d3e85b7a35/68747470733a2f2f7261772e6769746875622e636f6d2f6d616b6572616d656e2f526f756e646564496d616765566965772f6d61737465722f73637265656e73686f742e706e67