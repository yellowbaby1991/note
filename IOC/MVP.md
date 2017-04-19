* [概述](#概述)
* [MVP的开发步骤](#mvp的开发步骤)
* [一个MVP登陆实现的例子](#一个mvp登陆实现的例子)

### 概述
　MVP是专门为了优化Android开发的一种设计模式，核心思想是将Activity中的UI逻辑抽象成View接口，把业务逻辑抽象成Presenter接口， Model层用来处理数据，如访问网络和数据库

> 类图

![enter description here][1]

> MVP的优势

 1. 分离了视图逻辑和业务逻辑，降低了耦合
 2. Activity只处理生命周期的任务，代码变得更加简洁
 3. 视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去，提高代码的可阅读性
 4. Presenter被抽象成接口，可以有多种具体的实现，所以方便进行单元测试
 5. 把业务逻辑抽到Presenter中去，避免后台线程引用着Activity导致Activity的资源无法被系统回收从而引起内存泄露和OOM



### MVP的开发步骤

 1. 创建IPresenter接口，把所有业务逻辑的接口都放在这里，并创建它的实现PresenterCompl（在这里可以方便地查看业务功能，由于接口可以有多种实现所以也方便写单元测试）
 2. 创建IView接口，把所有视图逻辑的接口都放在这里，其实现类是当前的Activity/Fragment
 3. Activity里包含了一个IPresenter，而PresenterCompl里又包含了一个IView并且依赖了Model。Activity里只保留对IPresenter的调用，其它工作全部留到PresenterCompl中实现

### 一个MVP登陆实现的例子

 1. 添加常用工具包，logger，butterknife

> build.gradle

``` gradle
    compile 'com.orhanobut:logger:1.15'
    compile 'com.jakewharton:butterknife:8.5.1'
    annotationProcessor 'com.jakewharton:butterknife-compiler:8.5.1'
```
 2. model层

> User.java

``` java
public class User {
    private String name;
    private String password;

    public User(String name, String password) {
        this.name = name;
        this.password = password;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

 3. view层
 
> ILoginView.java

``` java
public interface ILoginView {
    public void onLoginResult(boolean result);
}

```
> LoginActivity.java

``` java
public class LoginActivity extends AppCompatActivity implements ILoginView {

    @BindView(R.id.username_et)
    EditText mUsernameEt;
    @BindView(R.id.password_et)
    EditText mPasswordEt;
    @BindView(R.id.login_bt)
    Button mLoginBt;

    ILoginPresenter loginPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        loginPresenter = new LoginPresenterCompl(this);
    }

    @Override
    public void onLoginResult(boolean result) {
        if (result) {
            Toast.makeText(this, "Login Success", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "Login Fail", Toast.LENGTH_SHORT).show();
        }
    }

    @OnClick(R.id.login_bt)
    public void login() {
        loginPresenter.doLogin(mUsernameEt.getText().toString(), mPasswordEt.getText().toString());
    }

}

```

> activity_main.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    android:paddingTop="@dimen/activity_vertical_margin">


    <EditText
        android:id="@+id/username_et"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="用户名" />

    <EditText
        android:id="@+id/password_et"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="密码" />

    <Button
        android:id="@+id/login_bt"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="登陆" />
    
</LinearLayout>

```


 4. presenter层

> ILoginPresenter.java

``` java
public interface ILoginPresenter {
    void doLogin(String name,String password);
}
```
> LoginPresenterCompl.java

``` java
public class LoginPresenterCompl implements ILoginPresenter {

    ILoginView mILoginView;
    Handler mHandler;

    public LoginPresenterCompl(ILoginView iLoginView) {
        this.mILoginView = iLoginView;
        mHandler = new Handler(Looper.getMainLooper());
    }

    private User findFromServer() {
        return new User("yellowbaby", "123456");
    }

    @Override
    public void doLogin(String name, String password) {
        User user = findFromServer();
        final boolean result = user.getName().equals(name) && user.getPassword().equals(password);
        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                mILoginView.onLoginResult(result);
            }
        }, 1000);

    }
}
```

  [1]: https://segmentfault.com/image?src=http://7xih5c.com1.z0.glb.clouddn.com/15-10-12/94032090.jpg&objectId=1190000003927200&token=62cb9888184d6fe02a4b3ae814ca17e8