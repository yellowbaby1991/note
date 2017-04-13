### 简介
　[Ｇlide][1]是Google员工的开源项目，Google I/O上被推荐使用的图片加载框架
 
### 配置方法

``` xml
dependencies {
  compile 'com.github.bumptech.glide:glide:3.7.0'
  compile 'com.android.support:support-v4:19.1.0'
}
```

### 使用方法

``` java
// For a simple view:
@Override public void onCreate(Bundle savedInstanceState) {
  ...
  ImageView imageView = (ImageView) findViewById(R.id.my_image_view);

  Glide.with(this).load("http://goo.gl/gEgYUd").into(imageView);
}

// For a simple image list:
@Override public View getView(int position, View recycled, ViewGroup container) {
  final ImageView myImageView;
  if (recycled == null) {
    myImageView = (ImageView) inflater.inflate(R.layout.my_image_view, container, false);
  } else {
    myImageView = (ImageView) recycled;
  }

  String url = myUrls.get(position);

  Glide
    .with(myFragment)
    .load(url)
    .centerCrop()
    .placeholder(R.drawable.loading_spinner)
    .crossFade()
    .into(myImageView);

  return myImageView;
}
```

### 常用技巧

 1. 禁止内存缓存
 
``` java
.skipMemoryCache(true);
```

 2. 清除缓存
 
``` java
Glide.get(context).clearMemory();
```

 3. 禁止磁盘缓存
 
``` java
.diskCacheStrategy(DiskCacheStrategy.NONE)
```

 4. 先显示缩略图，再显示原图
 
 5. 1

  [1]: https://github.com/bumptech/glide