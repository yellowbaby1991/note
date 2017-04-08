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



#### 短信拦截器

 1. 创建广播接收器

``` java
public class SmsReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Bundle bundle = intent.getExtras();
        Object[] pdus = (Object[]) bundle.get("pdus");
        for (Object pdu : pdus) {
            SmsMessage sms = SmsMessage.createFromPdu((byte[]) pdu);
            String address = sms.getDisplayOriginatingAddress();//发信人
            String smsBody = sms.getMessageBody();//短信内容
            if (smsBody.equals("nihao!")) {
                abortBroadcast();//拦截短信
            }
        }
    }
}
```

 2. 在AndroidManifest.xml中配置该广播接收器
 
``` xml
        <receiver android:name="com.yellow.SmsReceiver">
            <!-- 设置优先级 -->
            <intent-filter android:priority="1000">
                <action android:name="android.provider.Telephony.SMS_RECEIVED" />
            </intent-filter>
        </receiver>
```

 3. 放开去读短信的权限

``` xml
<uses-permission android:name="android.permission.RECEIVE_SMS"/>
```


### 自定义广播
#### 动态注册接收器接受无序广播

``` java
public class MainActivity extends AppCompatActivity {

    private DynamicReceiver dynamicReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //创建拦截器
        IntentFilter filter = new IntentFilter();
        filter.addAction("DynamicReceiver");
        dynamicReceiver = new DynamicReceiver();

        //注册接收器
        registerReceiver(dynamicReceiver, filter);
    }

    //发送名字为DynamicReceiver的无序广播，系统会根据配置的IntentFilter找接收器
    public void sendDynamic(View view) {
        Intent intent = new Intent();
        intent.setAction("DynamicReceiver");
        intent.putExtra("name", "黄贝");
        sendBroadcast(intent);
    }

    //注销接收器
    @Override
    protected void onPause() {
        super.onPause();
        unregisterReceiver(dynamicReceiver);
    }

    //自定义动态广播接收器
    class DynamicReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "动态广播：" + intent.getStringExtra("name"), Toast.LENGTH_SHORT).show();
        }
    }
}
```

#### 静态注册接收器接受无序广播

 1. 创建接收器
 
``` java
//自定义动态广播接收器
public class DynamicReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "动态广播：" + intent.getStringExtra("name"), Toast.LENGTH_SHORT).show();
    }

}
```

 2. AndroidManifest.xml中静态注册

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="app.yellow.broadcastreceiver">

    <application>
       ...

        <receiver android:name=".DynamicReceiver">
            <intent-filter>
                <action android:name="DynamicReceiver"></action>
            </intent-filter>
        </receiver>
    </application>

</manifest>
```


 3. sendBroadcast发送广播
 

``` stylus
enter code here
```
