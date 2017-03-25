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











  [1]: ./images/download.gif "download"
