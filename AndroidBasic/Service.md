### 服务是什么
　　服务是Android中实现程序后台运行的解决方案，非常适合去执行那些不需要和用户交互并且还要求长期运行的任务
  
### 服务的基本用法
#### 创建和销毁服务

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

#### 活动和服务通信

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

#### startService和bindService的区别
 
 1.生命周期的区别
- 执行startService方法，service会经历onCreate->onStartCommand，而执行bindService方法onCreate之后执行的是onBind方法
- startService方法启动的服务是长期在后台运行的，和原来的组件没什么关系了，bindService会随着绑定组件的销毁而销毁