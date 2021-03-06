* [服务](#服务)
	* [服务是什么](#服务是什么)
	* [服务的基本用法](#服务的基本用法)
		* [创建和销毁服务](#创建和销毁服务)
			* [活动和服务通信](#活动和服务通信)
			* [startService和bindService的区别](#startservice和bindservice的区别)
			* [使用Service+okhttp实现后台断点下载](#使用serviceokhttp实现后台断点下载)
			* [音乐播放器](#音乐播放器)
	* [AIDL](#aidl)
		* [AIDL是什么？](#aidl是什么)
		* [Android Studio下使用AIDL](#android-studio下使用aidl)
		* [AIDL调用流程](#aidl调用流程)
		* [不依赖AIDL实现IPC](#不依赖aidl实现ipc)
		* [模拟远程支付业务](#模拟远程支付业务)
	* [IntentService](#intentservice)
		* [什么是IntentService](#什么是intentservice)
		* [IntentService的用法](#intentservice的用法)
		* [IntentService的原理](#intentservice的原理)

# 服务
## 服务是什么
　　服务是Android中实现程序后台运行的解决方案，非常适合去执行那些不需要和用户交互并且还要求长期运行的任务
  
## 服务的基本用法 
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
 
 1. 生命周期的区别
	- 执行startService方法，service会经历onCreate->onStartCommand，而执行bindService方法onCreate之后执行的是onBind方法
	- startService方法启动的服务是长期在后台运行的，和原来的组件没什么关系了，bindService会随着绑定组件的销毁而销毁
	- 多次调用startService，onStartCommand会多次调用，但是多次执行bindService时，onBind方法并不会被多次调用
	
 2. 生命周期图对比
 
 ![enter description here][1]


#### 使用Service+okhttp实现后台断点下载 

　　[Service+okhttp实现后台断点下载][2]


#### 音乐播放器

 1. 将待播放的音乐导入Genymotion模拟器的mnt/shell/emulated/0/目录下
 2. 读取该目录下所有MP3格式的文件名显示在ListView上
 
``` java
public class MainActivity extends AppCompatActivity {
    private ListView mMusicLv;
    private List<String> mMusicPaths = new ArrayList<>();
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        readMusicFromSD();
        initMusicLv();
    }
	//读取SD卡目录下的所有文件并且筛选出后缀为MP3的
    private void readMusicFromSD() {
        if (BaseUtils.isSDCardHere()) {
            File[] files = BaseUtils.getSDFile().listFiles();
            for (File file : files) {
                String absolutePath = file.getAbsolutePath();
                if (absolutePath.endsWith(".mp3")) {
                    mMusicPaths.add(absolutePath);
                }
            }
        }
    }
	
	//给ListView配置数据
    private void initMusicLv() {
        mMusicLv = (ListView) findViewById(R.id.music_lv);
        ArrayAdapter adapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, android.R.id.text1, mMusicPaths);//根据数据源获得适配器
        mMusicLv.setAdapter(adapter);// 给ListView设置适配器
    }	
}

```

``` java
public class BaseUtils {

    //判断是否有SD卡
    public static boolean isSDCardHere() {
        return Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED);
    }

    //得到SD卡的根目录
    public static File getSDFile() {
        return Environment.getExternalStorageDirectory();
    }

}
```

 3. 创建播放服务
 
``` java
public class MusicService extends Service {

    private MediaPlayer mMediaPlayer;
    private int mCurrentPosition;//当前播放的位置

    private class MusicAgent extends Binder implements IMusicService {
        @Override
        public void callPlayMusic(List<String> filePaths, int postion) {
            playMusic(filePaths, postion);
        }
    }

    private void playMusic(final List<String> filePaths, int postion) {
        mCurrentPosition = postion;
        if (mMediaPlayer == null) {
            mMediaPlayer = new MediaPlayer();
        }
        try {
            mMediaPlayer.reset();
            mMediaPlayer.setDataSource(filePaths.get(mCurrentPosition));
            mMediaPlayer.prepare();
            mMediaPlayer.start();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return new MusicAgent();
    }
}
```

``` java
//暴露给客户端的接口
public interface IMusicService {
    public void callPlayMusic(List<String> filePaths, int postion);
}
```

 4. Activity开启服务后绑定
 
``` java
public class MainActivity extends AppCompatActivity {

    private IMusicService mMusicService;
    private ServiceConnection mConn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mMusicService = (IMusicService) service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        initMusicService();
    }	
	
    private void initMusicService() {
        Intent intent = new Intent(this, MusicService.class);
        startService(intent);
        bindService(intent, mConn, BIND_AUTO_CREATE);
    }	
	
}
```
 5. 添加item点击事件调用服务的play方法
 
``` java
public class MainActivity extends AppCompatActivity {
    ...
    private void initMusicLv() {
        ...
        mMusicLv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                mMusicService.callPlayMusic(mMusicPaths, position);
            }
        });
    }
	
}
```

 6. 添加menu，实现三种播放模式，根据SharedPreferences中保存的值来进行不同的播放操作
 

``` xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/stop_when_over"
        android:title="播完停止" />
    <item
        android:id="@+id/single_loop"
        android:title="单曲循环" />
    <item
        android:id="@+id/all_loop"
        android:title="全部循环" />
</menu>
```

``` java
public class MainActivity extends AppCompatActivity {
    ...
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater menuInflater = getMenuInflater();
        menuInflater.inflate(R.menu.main, menu);
        return true;
    }
	
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        SharedPreferences sp = getSharedPreferences("music", MODE_PRIVATE);
        SharedPreferences.Editor editor = sp.edit();
        switch (item.getItemId()) {
            case R.id.stop_when_over:
                editor.putInt("palyMode", 1);
                break;
            case R.id.single_loop:
                editor.putInt("playMode", 2);
                break;
            case R.id.all_loop:
                editor.putInt("playMode", 3);
                break;
        }
        editor.commit();
        return true;
    }
}
```

``` java
public class MusicService extends Service {
    
    private void playMusic(final List<String> filePaths, int postion) {
        try {
            ...
            mMediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                @Override
                public void onCompletion(MediaPlayer mp) {
                    SharedPreferences sp = getSharedPreferences("music", MODE_PRIVATE);
                    int playMode = sp.getInt("playMode", 1);
                    if (playMode == 2) {//单曲循环
                        playMusic(filePaths, mCurrentPosition);
                    } else if (playMode == 3) {//列表循环
                        mCurrentPosition++;
                        if (mCurrentPosition == filePaths.size()) {
                            mCurrentPosition = 0;
                        }
                        playMusic(filePaths, mCurrentPosition);
                    }
                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
 
 7. 添加播放时候展示的通知栏
 

``` java
public class MusicService extends Service {

    private void playMusic(final List<String> filePaths, int postion) {
          notification(filePaths.get(postion));	
          ...
          //播放操作
	}
	
    //安卓16之后显示通知建议采用builder
    private void notification(String musicName) {
        NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        Notification.Builder builder = new Notification.Builder(this);
        builder.setSmallIcon(R.mipmap.ic_launcher); //设置图标
        builder.setContentTitle("音乐播放"); //设置标题
        builder.setContentText("当前播放:"+musicName); //消息内容
        builder.setWhen(System.currentTimeMillis()); //发送时间
        Intent intent = new Intent(this, MainActivity.class);
        PendingIntent contentIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_ONE_SHOT);
        builder.setContentIntent(contentIntent);
        Notification notification = null;
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN) {
            notification = builder.build();
        }
        manager.notify(0, notification); // 通过通知管理器发送通知
    }	
}
```

 8. 添加退出APP选项

``` java
public interface IMusicService {
   ...
    public void callStopPlay();
}
```

``` java
public class MusicService extends Service {
    ...
    private class MusicAgent extends Binder implements IMusicService {
        ...
        @Override
        public void callStopPlay() {
            stopPlay();
        }
    }
    //关闭音乐播放以及取消通知
    private void stopPlay() {
        if (mMediaPlayer != null){
            if (mMediaPlayer.isPlaying()){
                mMediaPlayer.stop();
            }
            mMediaPlayer.release();
            mMediaPlayer = null;
        }
        NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        manager.cancelAll();
    }	
}
```
 
``` java
<menu xmlns:android="http://schemas.android.com/apk/res/android">
   ...
    <item
        android:id="@+id/logout_app"
        android:title="退出应用" />
</menu>
```

``` java
public class MainActivity extends AppCompatActivity {
    ...
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            ...
            case R.id.logout_app:
                mMusicService.callStopPlay();//取消音乐播放和通知显示
                unbindService(mConn);//解绑服务
                Intent intent = new Intent(this,MusicService.class);
                stopService(intent);//结束服务
                finish();//关闭界面
        }
        return true;
    }
}
```

## AIDL
### AIDL是什么？
　　AIDL是Android的一种IPC（跨进程通信）的方式，内部原理是Binder
### Android Studio下使用AIDL  

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

 4. 将该服务配置成remote，意思是可以多进程
 
``` xml
<service
	android:name=".MyServer"
	android:process=":remote">
	<intent-filter>
		<action android:name="com.yellow.MyServer"></action>
	</intent-filter>
</service>
```

 5. 再建一个Client工程作为客户端
 6. 把aidl文件夹的内容完全拷贝过去，包名也一致
 7. 客户端的activity如下
 
``` java
public class MainActivity extends AppCompatActivity {

    IMyAidlInterface mStub;

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mStub = IMyAidlInterface.Stub.asInterface(service);
            try {
                int value = mStub.add(1, 8);
                Toast.makeText(MainActivity.this, value + "", Toast.LENGTH_SHORT).show();
                Log.i("TAG", value + "");
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent();
        intent.setAction("com.yellow.MyServer");
        bindService(intent, serviceConnection, BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(serviceConnection);
    }
}
```
8. 然后先启动服务端，再启动客户端就可以实现远程调用了

### AIDL调用流程

 1. 看看由AIDL生成的java文件，可以发现一共只有三个东西，接口方法，Stub类给服务端用，Proxy给客户端用
 
``` java
public interface IMyAidlInterface extends android.os.IInterface {

    //服务端端实际上拿到的对象
    public static abstract class Stub extends android.os.Binder implements app.yellow.myaidldemoserver.IMyAidlInterface {
        ...
        //客户端调用这个方法得到Proxy对象
        public static app.yellow.myaidldemoserver.IMyAidlInterface asInterface(android.os.IBinder obj) {
            ...
            return new app.yellow.myaidldemoserver.IMyAidlInterface.Stub.Proxy(obj);
        }
        //服务端使用该方法将结果传回客户端
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_add: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    int _arg1;
                    _arg1 = data.readInt();
                    int _result = this.add(_arg0, _arg1);
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        //客户端实际上拿到的对象
        private static class Proxy implements app.yellow.myaidldemoserver.IMyAidlInterface {
            ...
            //客户端实际执行的add方法，发送请求给客户端
            @Override
            public int add(int arg1, int arg2) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(arg1);
                    _data.writeInt(arg2);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }
    //接口
    public int add(int arg1, int arg2) throws android.os.RemoteException;
}
```


 2. 客户端实际拿到的对象其实是Proxy
 
``` java
public class MainActivity extends AppCompatActivity {
    ...
    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mStub = IMyAidlInterface.Stub.asInterface(service);
            ...
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
}
```


``` java
public interface IMyAidlInterface extends android.os.IInterface {
        ...
        public static app.yellow.myaidldemoserver.IMyAidlInterface asInterface(android.os.IBinder obj) {
            ...
            return new app.yellow.myaidldemoserver.IMyAidlInterface.Stub.Proxy(obj);
        }
}
```


 3. 所以实际上执行的方法是Proxy中的add方法
 
``` java
public interface IMyAidlInterface extends android.os.IInterface {
    ...
    public static abstract class Stub extends android.os.Binder implements app.yellow.myaidldemoserver.IMyAidlInterface {
	    ...
        private static class Proxy implements app.yellow.myaidldemoserver.IMyAidlInterface {	
		    //这就是客户端实际上执行的代码
            @Override
            public int add(int arg1, int arg2) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(arg1);
                    _data.writeInt(arg2);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }		
		}
	}
}
```


 4. 通过mRemote.transact这个方法向服务端发送了请求，服务端的onTransact用来处理请求
 
``` java
public interface IMyAidlInterface extends android.os.IInterface {
    ...
    public static abstract class Stub extends android.os.Binder implements app.yellow.myaidldemoserver.IMyAidlInterface {
	    ....
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
               ....
                case TRANSACTION_add: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    int _arg1;
                    _arg1 = data.readInt();
                    int _result = this.add(_arg0, _arg1);//调用服务端的add方法将结果发回
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }	
	}
}	
```

 5. 所以AIDL实际上本质就是Binder的跨进程通信，工作流程图如下

![enter description here][3]

### 不依赖AIDL实现IPC

　　从上面的分析可以看出整体流程就是：客户端调用Proxy的add方法发送请求，服务端调用onTransact接受请求调用服务端真正的add方法处理结果后发回客户端，我们现在看看不通过ADIL来实现同样的功能
  
  
 1. 服务端Service重写onTransact处理DESCRIPTOR为"CalcPlusService"的请求
 
``` java
public class MyServer extends Service {

    private static final String DESCRIPTOR = "CalcPlusService";

    private class MyBinder extends Binder {
        @Override
        protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
            data.enforceInterface(DESCRIPTOR);
            int _arg0;
            _arg0 = data.readInt();
            int _arg1;
            _arg1 = data.readInt();
            int _result = _arg0 + _arg1;
            reply.writeNoException();
            reply.writeInt(_result);
            return true;
        }
    }

    public MyServer() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        return new MyBinder();
    }
}
```


 2. 客户端直接发送请求，这样就实现了不依赖AIDL纯粹使用Binder跨进程通信
 
``` java
public class MainActivity extends AppCompatActivity {

    private IBinder mPlusBinder;

    private ServiceConnection serviceConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mPlusBinder = service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    public void add(View view) {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        int _result;
        try {
            _data.writeInterfaceToken("CalcPlusService");
            _data.writeInt(50);
            _data.writeInt(12);
            mPlusBinder.transact(0x110, _data, _reply, 0);
            _reply.readException();
            _result = _reply.readInt();
            Toast.makeText(this, _result + "", Toast.LENGTH_SHORT).show();

        } catch (RemoteException e) {
            e.printStackTrace();
        } finally {
            _reply.recycle();
            _data.recycle();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent();
        intent.setAction("MyServer");
        bindService(intent, serviceConnection, BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(serviceConnection);
    }
}
```


  [1]: ./images/Service%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE.jpg "Service生命周期图"
  [2]: https://github.com/yellowbaby1991/note/blob/master/HTTP/OkHttp%E5%AE%9E%E7%8E%B0%E6%96%AD%E7%82%B9%E4%B8%8B%E8%BD%BD.md
  [3]: ./images/Binder%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE.png "Binder工作流程图"
### 模拟远程支付业务

 1. 服务端main/aidl/文件夹下创建IAlipayService.aidl接口文件
 
``` java
package app.yellow.alipay;

interface IAlipayService {
	int callSafePay(String account, String pwd, double money,
			long currenTimeMiles);
}

```

 2. 服务端端创建AlipayService

``` java
public class AlipayService extends Service {

    public AlipayService() {
    }

    private class AlipayAgent extends IAlipayService.Stub {

        @Override
        public int callSafePay(String account, String pwd, double money, long currenTimeMiles) throws RemoteException {
            return safePay(account, pwd, money, currenTimeMiles);
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return new AlipayAgent();
    }

    public int safePay(String account, String pwd, double money, long currentTimeMiles) {
        if (!account.equals("zhangsan") || !pwd.equals("123")) {
            return 404;
        }
        if (money > 10) {// 如果支付的金额大于10块钱
            return 500;
        }
        return 200;
    }
}
```

 3. 给AlipayService配置IntentFilter
 
``` xml
<service android:name=".AlipayService">
	<intent-filter>
		<action android:name="com.yellow.ALIPAY"></action>
	</intent-filter>
</service>
```

 4. 客户端同样拷贝aidl文件到相同位置
 5. 绑定连接后远程调用

``` java
public class MainActivity extends AppCompatActivity {

    private IAlipayService iAlipayService;

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iAlipayService = IAlipayService.Stub.asInterface(service);
            mockPay();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void pay(View view) {
        Intent service = new Intent("com.yellow.ALIPAY");
        bindService(service, serviceConnection, BIND_AUTO_CREATE);
    }

    protected void mockPay() {
        try {
            int result = iAlipayService.callSafePay("zhangsan", "123", 20, System.currentTimeMillis());
            switch (result) {
                case 200:
                    Toast.makeText(MainActivity.this, "支付成功", Toast.LENGTH_SHORT).show();
                    break;
                case 404:
                    Toast.makeText(MainActivity.this, "账号密码错误", Toast.LENGTH_SHORT).show();
                    break;
                case 500:
                    Toast.makeText(MainActivity.this, "余额不足", Toast.LENGTH_SHORT).show();
                    break;
            }

        } catch (RemoteException e) {
            e.printStackTrace();
        }

    }
}
```

## IntentService
### 什么是IntentService

 1. IntentService是一种特殊的Service，它可以执行多个后台任务
 2. IntentService的任务默认在子线程执行
 3. IntentService完成任务后会自动关闭
 
### IntentService的用法

``` java
public class MyIntentService extends IntentService {

    final static String TAG = "MyIntentService";

    public MyIntentService() {
        super("service");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        Log.i(TAG, "begin onHandleIntent() in " + this);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Log.i(TAG, "end onHandleIntent() in " + this);
    }

    public void onDestroy() {
        super.onDestroy();
        Log.i(TAG, this + " is destroy");
    }
}
```

启动方式和普通service一样

``` java
Intent intent=new Intent(this,MyIntentService.class);
startService(intent);
startService(intent);
startService(intent);
```

会等到所有任务都执行完之后该服务自动onDestroy


### IntentService的原理

 1. 先看onCreate方法，内部创建了一个HandlerThread，用它的Looper对象构造一个mServiceHandler，用来给子线程通信
 
``` java
public abstract class IntentService extends Service {
	...
    //第一次启动的时候调用
    @Override
    public void onCreate() {

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
  	...
}
```


 2. 启动的时候调用onStart，调用mServiceHandler对象sendMessage
 
``` java
public abstract class IntentService extends Service {
	...
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
  	...
}
```

 3. 接受后，在子线程中执行任务
 
``` java
public abstract class IntentService extends Service {
	...
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);//子类实现
            stopSelf(msg.arg1);
        }
    }
    protected abstract void onHandleIntent(@Nullable Intent intent);
    ...
}
```

 4. 执行完了之后，自动杀死服务
 

``` java
public abstract class Service
  	...
    //和stopSelf()不同，这个方法会等最近启动的所有服务都关闭才关闭，startId就是次数
    public final void stopSelf(int startId) {
        if (mActivityManager == null) {
            return;
        }
        try {
            mActivityManager.stopServiceToken(
                    new ComponentName(this, mClassName), mToken, startId);
        } catch (RemoteException ex) {
        }
    }
	...
}
```


