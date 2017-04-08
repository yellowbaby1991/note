* [服务是什么](#服务是什么)
* [创建和销毁服务](#创建和销毁服务)
* [活动和服务通信](#活动和服务通信)
* [startService和bindService的区别](#startservice和bindservice的区别)
* [使用Service+okhttp实现后台断点下载](#使用serviceokhttp实现后台断点下载)

### 服务是什么
　　服务是Android中实现程序后台运行的解决方案，非常适合去执行那些不需要和用户交互并且还要求长期运行的任务
  
### 创建和销毁服务

 1. 创建一个Service的子类，重写下面几个方法
 
``` java
public class MyService extends Service {

    //只会在服务第一次创建的时候调用
    @Override
    public void onCreate() {
        super.onCreate();
    }
	
    //每次启动服务都会调用
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

}
```


 2. Activity中调用startActivity和stopActivity来开启或者销毁服务
 
``` java
public class MainActivity extends AppCompatActivity {
    ...
	//启动服务
    public void startService(View view) {
        Intent startIntent = new Intent(this, MyService.class);
        startService(startIntent);
    }

    //停止服务
    public void stopService(View view) {
        Intent stopIntent = new Intent(this, MyService.class);
        stopService(stopIntent);
    }

}
```

### 活动和服务通信

 1. 在service中定义一个提供给Activity的代理（Binder）

``` java
public class MyService extends Service {

    private DownloadBinder mBinder = new DownloadBinder();

    //提供给Activity的小秘书
    class DownloadBinder extends Binder {
        public void startDownload() {
            //下载逻辑
        }
    }

    //将小秘书返回给Activity
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    ...
}
```


 2. Activity中创建一个Connection来获得代理

``` java
public class MainActivity extends AppCompatActivity {

    private MyService.DownloadBinder downloadBinder;

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        //第二个参数就是service传来的代理
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder) service;
            downloadBinder.startDownload();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    ...
	
    public void startService(View view) {
        Intent startIntent = new Intent(this, MyService.class);
        bindService(startIntent, connection, BIND_AUTO_CREATE);//绑定服务
    }

    public void stopService(View view) {
        Intent stopIntent = new Intent(this, MyService.class);
        unbindService(connection);   //停止服务
    }

}
```

### startService和bindService的区别
 
 1. 生命周期的区别
	- 执行startService方法，service会经历onCreate->onStartCommand，而执行bindService方法onCreate之后执行的是onBind方法
	- startService方法启动的服务是长期在后台运行的，和原来的组件没什么关系了，bindService会随着绑定组件的销毁而销毁
	- 多次调用startService，onStartCommand会多次调用，但是多次执行bindService时，onBind方法并不会被多次调用
	
 2. 生命周期图对比
 
 ![enter description here][1]


### 使用Service+okhttp实现后台断点下载 

　　[Service+okhttp实现后台断点下载][2]


  [1]: ./images/Service%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE.jpg "Service生命周期图"
  [2]: https://github.com/yellowbaby1991/note/blob/master/HTTP/OkHttp%E5%AE%9E%E7%8E%B0%E6%96%AD%E7%82%B9%E4%B8%8B%E8%BD%BD.md
### AIDL
#### AIDL是什么？
　　AIDL是Android的一种IPC（跨进程通信）的方式，内部原理是Binder
#### Android Studio下使用AIDL  

 1. 新建一个Server项目作为服务端，在main下建立一个文件夹aidl，然后建立一个IMyAidlInterface.aidl文件
 
``` java
package app.yellow.myaidldemoserver;

interface IMyAidlInterface {
    int add(int arg1, int arg2);
}

```


 2. 点击编译按钮，app/build/generated/source/aidl/debug目录下会生成一个与IMyAidlInterface.aidl文件同样包名的一个文件
 
 3. 建立一个service

``` java
public class MyServer extends Service {

    IMyAidlInterface.Stub mStub = new IMyAidlInterface.Stub() {
        @Override
        public int add(int arg1, int arg2) throws RemoteException {
            return arg1 + arg2;
        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        return mStub;
    }
}
```

 4.将该服务配置成remote，意思是可以多进程
 
``` xml
<service
	android:name=".MyServer"
	android:process=":remote">
	<intent-filter>
		<action android:name="com.yellow.MyServer"></action>
	</intent-filter>
</service>
```

 5.再建一个Client工程作为客户端
 6.把aidl文件夹的内容完全拷贝过去，包名也一致
 7.客户端的activity如下