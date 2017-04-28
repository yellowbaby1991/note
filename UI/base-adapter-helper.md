### 概述
　[base-adapter-helper][1] 是BaseAdapter ViewHolder模板的抽象，可以快速实现Adapter，让我们从大量重复代码中解脱出来
 
### 配置方法

``` xml
dependencies {
    compile'com.joanzapata.android:base-adapter-helper:1.1.11'
}
```

### 使用方法

 1. 创建model类

> ModuleBean.java

``` java
public class ModuleBean {
    private String modulename;
    private String imgurl;
    private String description;
    ...//get & set
}
```
 2. 创建布局
> item.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/name_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <ImageView
        android:id="@+id/img_iv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/description_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

 3. 快速创建适配器
 
> MainActivity.java

``` java
public class MainActivity extends AppCompatActivity {

    private ListView mListView;
    private List<ModuleBean> mModuleBeanList;
    private QuickAdapter<ModuleBean> mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mListView = (ListView) findViewById(R.id.lv);

        mModuleBeanList = DataUtils.getAdapterData();

        if (mAdapter == null) {
            mAdapter = new QuickAdapter<ModuleBean>(this, R.layout.item, mModuleBeanList) {
                @Override
                protected void convert(BaseAdapterHelper helper, ModuleBean item) {
                    helper.setText(R.id.name_tv, item.getModulename()).setImageUrl(R.id.img_iv, item.getImgurl()).setText(R.id.description_tv, item.getDescription());
					           helper.setOnClickListener(R.id.iv_delete, new View.OnClickListener() {//添加监听}
                }
            };
        }
        mListView.setAdapter(mAdapter);
    }
}
```


  [1]: https://github.com/JoanZapata/base-adapter-helper