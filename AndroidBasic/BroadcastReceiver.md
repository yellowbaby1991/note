### 广播的概念

　　系统很多消息需要通过广播去通知给每一个应用程序，比如电量不足，网络状态的变化，SD卡被拔出插入，有新的apk被安装等等，如果某个应用程序关心某个广播，就需要注册一个广播接收器（BroadcastReceiver）
  
### 接受常用的系统广播

#### SD卡状态监听

 1. 创建广播接收器
 
``` java
public class SDCardReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		String action = intent.getAction();
		if (action.equals(Intent.ACTION_MEDIA_MOUNTED)) {
			//安装处理			
		}else if (action.equals(Intent.ACTION_MEDIA_UNMOUNTED)) {
			//卸载处理
		}
	}

}
```

 2. 在AndroidManifest.xml中配置该广播接收器
 
``` java
        <receiver android:name="com.yellow.SDCardReceiver">
            <intent-filter>
                <action android:name="android.intent.action.MEDIA_MOUNTED" />
                <action android:name="android.intent.action.MEDIA_UNMOUNTED" />
                <!-- 广播类型 -->
                <data android:scheme="file" />
            </intent-filter>
        </receiver>			
```


 3. 1