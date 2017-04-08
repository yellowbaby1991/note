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

#### 