### 简介
　[Butter Knife][1]是大名鼎鼎的JakeWharton贡献的IOC框架，优势如下:
  
 1. 强大的View绑定和Click事件处理功能，简化代码，提升开发效率
 2. 方便的处理Adapter里的ViewHolder绑定问题
 3. 运行时不会影响APP效率，使用配置方便
 4. 代码清晰，可读性强

### 配置方法

``` xml
dependencies {
  compile 'com.jakewharton:butterknife:8.5.1'
  annotationProcessor 'com.jakewharton:butterknife-compiler:8.5.1'
}
```

### 使用方法
　[官方使用文档][2]

 1. 使用BindView取代繁琐的findViewById

``` java
class ExampleActivity extends Activity {
  @BindView(R.id.title) TextView title;
  @BindView(R.id.subtitle) TextView subtitle;
  @BindView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

 2. 也可以绑定特定的资源，如string，color，drawable等

``` java
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}
```

 3. 在fragment中也可以使用BindView

  [1]: https://github.com/JakeWharton/butterknife
  [2]: http://jakewharton.github.io/butterknife/