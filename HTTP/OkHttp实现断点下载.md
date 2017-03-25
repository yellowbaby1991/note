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

 2. 在DownloadTask的doInBackground中根据任务的状态










  [1]: ./images/download.gif "download"