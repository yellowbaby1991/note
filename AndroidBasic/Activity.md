### 应用程序的国际化

 1. 使用@string配置取代layout中的硬编码
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/text" />
    
</RelativeLayout>

```

 2. values和values-en文件夹下建立对应的string.xml
 3. 