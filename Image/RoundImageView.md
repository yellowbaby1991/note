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


 2. 1

  [1]: https://github.com/vinc3m1/RoundedImageView
  [2]: https://camo.githubusercontent.com/ed1e075be6ed97fa9091d3702e9b96d3e85b7a35/68747470733a2f2f7261772e6769746875622e636f6d2f6d616b6572616d656e2f526f756e646564496d616765566965772f6d61737465722f73637265656e73686f742e706e67