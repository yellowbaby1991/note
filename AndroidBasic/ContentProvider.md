### 内容提供器是什么
　　ContentProvider主要用于不同程序间数据共享，是IPC的一种方式，提供了一套完整的机制运行跨进程通信，并且同时还可以保证被访问数据的安全性
  
### 自定义内容提供器
#### 自定义ContentProvider

 1. 创建DBOpenHelper
 
``` java
public class DBOpenHelper extends SQLiteOpenHelper {

    public DBOpenHelper(Context context) {
        super(context, DbConst.DB_NAME, null, DbConst.DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("create table " + DbConst.TABLE_NAME + " ("
                + DbConst._ID + " integer primary key autoincrement,"
                + DbConst.COLUMN_NAME + " varchar(10),"
                + DbConst.COLUMN_PHONE + " varchar(13));");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}
```


 2. 1

#### 自定义ContentResolver