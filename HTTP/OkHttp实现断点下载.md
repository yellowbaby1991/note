#### 需求
　　提交Http请求实现从服务器上下载一个文件，在通知栏显示下载进度，支持断点下载和任务取消，并且兼容Android6.0的动态权限获取
  
#### 效果
![enter description here][1]
  
#### 分析

 1. 网络请求使用OkHttp来实现，OkHttp很好的支持了断点下载
 2. 使用AsyncTask进行后台下载
 3. 使用Service服务启动Task防止任务被杀死

#### 细节

 1. 下载任务DownloadTask一共具有四种状态。下载成功，下载失败，下载暂停，下载取消
 
``` java
public class DownloadTask extends AsyncTask<String, Integer, Integer> {
    /**
     * 下载成功
     */
    public static final int TYPE_SUCCESS = 0;
    /**
     * 下载失败
     */
    public static final int TYPE_FAILED = 1;
    /**
     * 下载暂停
     */
    public static final int TYPE_PAUSED = 2;
    /**
     * 下载取消
     */
    public static final int TYPE_CANCELED = 3;
	...
}
```

 2. 在DownloadTask的doInBackground中根据任务的状态来判断是下载，暂停，还是取消

``` java
public class DownloadTask extends AsyncTask<String, Integer, Integer> {
     ...
    @Override
    protected Integer doInBackground(String... params) {
        InputStream is = null;
        RandomAccessFile savedFile = null;
        File file = null;
        try {
            long downloadedLength = 0;//记录已下载的文件长度
            String downloadUrl = params[0];//下载链接
            String fileName = downloadUrl.substring(downloadUrl.lastIndexOf("/"));//截取得到文件名
            String directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getPath();//得到本机存储目录
            file = new File(directory + fileName);//得到File的绝对路径
            if (file.exists()) {
                downloadedLength = file.length();
            }
            long contentLength = getContentLength(downloadUrl);
            if (contentLength == 0) {
                return TYPE_FAILED;
            } else if (contentLength == downloadedLength) {
                return TYPE_SUCCESS;
            }
            //使用OkHttp断点下载
            OkHttpClient client = new OkHttpClient();
            Request request = new Request.Builder().
                    addHeader("RANGE", "bytes=" + downloadedLength + "-").
                    url(downloadUrl).
                    build();
            Response response = client.newCall(request).execute();
            if (response != null) {
                is = response.body().byteStream();
                savedFile = new RandomAccessFile(file, "rw");
                savedFile.seek(downloadedLength);//找到已经下载的长度
                byte[] b = new byte[1024];
                int total = 0;
                int len;
                while ((len = is.read(b)) != -1) {
                    if (isCanceled) {//取消下载
                        return TYPE_CANCELED;
                    } else if (isPaused) {//暂停下载
                        return TYPE_PAUSED;
                    } else {
                        total += len;
                    }
                    savedFile.write(b, 0, len);//如果没有取消，没有暂停就继续下载
                    int progress = (int) ((total + downloadedLength) * 100 / contentLength);
                    publishProgress(progress);
                }
            }
            response.close();
            return TYPE_SUCCESS;//下载成功

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (is != null) {
                    is.close();
                }
                if (savedFile != null) {
                    savedFile.close();
                }
                if (isCanceled && file != null) {
                    file.delete();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return TYPE_FAILED;
    }
}
```

3. 在DownloadService中创造Binder对象，使用DownloadTask来执行下载任务

``` java
public class DownloadService extends Service {
	...
	private DownloadTask downloadTask;
	
	private DownloadBinder mBinder = new DownloadBinder();
	
	@Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
	//传给Activity使用的Binder对象
	class DownloadBinder extends Binder {
        //执行开始下载任务
        public void startDownload(String url) {
            if (downloadTask == null) {
                downloadUrl = url;
                downloadTask = new DownloadTask(listener);
                downloadTask.execute(downloadUrl);
                startForeground(1, getNotification("Downloading...", 0));
                Toast.makeText(DownloadService.this, "Downloading...", Toast.LENGTH_SHORT).show();
            }
        }
        //执行暂停下载任务
        public void pauseDownload() {
            if (downloadTask != null) {
                downloadTask.pauseDownload();
            }
        }
        //执行取消下载任务
        public void cancelDownload() {
            if (downloadTask != null) {
                downloadTask.cancelDownload();
            } else {
                if (downloadUrl != null) {
                    String fileName = downloadUrl.substring(downloadUrl.lastIndexOf("/"));
                    String directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getPath();
                    File file = new File(directory + fileName);
                    if (file.exists()) {
                        file.delete();
                    }
                    getNotificationManager().cancel(1);
                    stopForeground(true);
                    Toast.makeText(DownloadService.this, "Cancled", Toast.LENGTH_SHORT).show();
                }

            }
        }

    }
	...
}
```

4. 在MainActivity中绑定服务，得到服务传过来的binder对象后进行调用

``` java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
	...
	private DownloadService.DownloadBinder downloadBinder;
	
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (DownloadService.DownloadBinder) service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
    };
	    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		...
        Intent intent = new Intent(this, DownloadService.class);
        startService(intent);
        bindService(intent, connection, BIND_AUTO_CREATE);//绑定服务

    }
	
    @Override
    public void onClick(View v) {
        if (downloadBinder == null){
            return;
        }
		//调用binder对象执行下载任务
        switch (v.getId()){
            case R.id.start_download:
                String url = "http://192.168.87.2/2.pdf";
                downloadBinder.startDownload(url);
                break;
            case R.id.pause_download:
                downloadBinder.pauseDownload();
                break;
            case R.id.cancel_download:
                downloadBinder.cancelDownload();
                break;
            default:
        }
    }
}
```

5. 在MainActivity中动态判断是否具有权限，如果没有就动态申请
 











  [1]: ./images/download.gif "download"
