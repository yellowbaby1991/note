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

``` java
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }
}
```

 4. 在Adapter的ViewHolder 中使用BindView

``` java
public class MyAdapter extends BaseAdapter {
  @Override public View getView(int position, View view, ViewGroup parent) {
    ViewHolder holder;
    if (view != null) {
      holder = (ViewHolder) view.getTag();
    } else {
      view = inflater.inflate(R.layout.whatever, parent, false);
      holder = new ViewHolder(view);
      view.setTag(holder);
    }

    holder.name.setText("John Doe");
    // etc...

    return view;
  }

  static class ViewHolder {
    @BindView(R.id.title) TextView name;
    @BindView(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.bind(this, view);
    }
  }
}
```

 5. 使用BindViews绑定多个Id
 
``` java
public class MainActivity extends AppCompatActivity {

    @BindViews({ R.id.button1  , R.id.button2 ,  R.id.button3 })
    public List<Button> buttonList ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);

        ButterKnife.bind(this);

        buttonList.get( 0 ).setText( "hello 1 ");
        buttonList.get( 1 ).setText( "hello 2 ");
        buttonList.get( 2 ).setText( "hello 3 ");
    }
}
```

 6. 绑定监听事件
 
``` java
public class ButterknifeActivity extends AppCompatActivity {

    @OnClick(R.id.button1 )   //给 button1 设置一个点击事件
    public void showToast(){
        Toast.makeText(this, "is a click", Toast.LENGTH_SHORT).show();
    }

    @OnLongClick( R.id.button1 )    //给 button1 设置一个长按事件
    public boolean showToast2(){
        Toast.makeText(this, "is a long click", Toast.LENGTH_SHORT).show();
        return true  ;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_butterknife);

        //绑定activity
        ButterKnife.bind( this ) ;

    }
}
```


  [1]: https://github.com/JakeWharton/butterknife
  [2]: http://jakewharton.github.io/butterknife/