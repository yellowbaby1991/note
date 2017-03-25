#### 需求
　　提交Http请求实现从服务器上下载一个文件，在通知栏显示下载进度，支持断点下载和任务取消，并且兼容Android6.0的动态权限获取
  
#### 效果
![enter description here][1]
  
#### 分析

 1. 网络请求使用OkHttp来实现，OkHttp很好的支持了断点下载
 2. 使用AsyncTask进行后台下载
 3. 















  [1]: ./images/download.gif "download"