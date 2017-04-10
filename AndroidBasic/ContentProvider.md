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
 
``` java
public class DbConst implements BaseColumns {
    public static final int DB_VERSION = 1;
    public static final String DB_NAME = "contact.db";
    public static final String TABLE_NAME = "contactinfo";
    public static final String COLUMN_NAME = "name";
    public static final String COLUMN_PHONE = "phone";
}
```

 2. 创建ContentProvider，调用dbhelper访问数据库
 
``` java
public class MyProvider extends ContentProvider {

    private DBOpenHelper mHelper;
    private SQLiteDatabase mDb;

    private static final String AUTH0RITY = "com.yellow.cp.contacts";
    public static final int CONTACT_ALL = 1;
    public static final int CONTACT_SINGLE = 2;

    private static UriMatcher mUriMatcher;

    //为了安全性使用UriMatcher来匹配数据
    static {
        mUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        mUriMatcher.addURI(AUTH0RITY, "contacts", CONTACT_ALL);//匹配全部数据
        mUriMatcher.addURI(AUTH0RITY, "contacts/*", CONTACT_SINGLE); //匹配单行数据，*代表字符，#代表数字
    }

    @Override
    public boolean onCreate() {
        mHelper = new DBOpenHelper(getContext());
        mDb = mHelper.getReadableDatabase();
        return true;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        switch (mUriMatcher.match(uri)) {
            case CONTACT_ALL:
                long id = mDb.insert(DbConst.TABLE_NAME, null, values);
                return Uri.parse(uri.toString() + "/" + id);
            case CONTACT_SINGLE:
                return null;
        }
        return null;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        Cursor cursor = null;
        switch (mUriMatcher.match(uri)) {
            case CONTACT_ALL:
                cursor = mDb.query(DbConst.TABLE_NAME, projection, selection, selectionArgs, null, null, null);
                break;
            case CONTACT_SINGLE:
                String name = uri.getPathSegments().get(1);
                cursor = mDb.query(DbConst.TABLE_NAME, projection, "name=?", new String[]{name}, null, null, null);
                break;
        }
        return cursor;
    }


    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        switch (mUriMatcher.match(uri)) {
            case CONTACT_ALL:
                return mDb.update(DbConst.TABLE_NAME, values, selection, selectionArgs);
            case CONTACT_SINGLE:
                String name = uri.getPathSegments().get(1);
                return mDb.update(DbConst.TABLE_NAME, values, "name=?", new String[]{name});
        }
        return 0;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        switch (mUriMatcher.match(uri)) {
            case CONTACT_ALL:
                return mDb.delete(DbConst.TABLE_NAME, selection, selectionArgs);
            case CONTACT_SINGLE:
                String name = uri.getPathSegments().get(1);
                return mDb.delete(DbConst.TABLE_NAME, "name=?", new String[]{name});
        }
        return 0;
    }

    //用于告诉调用者返回数据的类型
    @Override
    public String getType(Uri uri) {
        switch (mUriMatcher.match(uri)) {
            case CONTACT_ALL:
                return "vnd.android.cursor.dir/vnd.com.yellow.cp.contacts";
            case CONTACT_SINGLE:
                return "vnd.android.cursor.item/vnd.com.yellow.cp.contacts";
        }
        return "";
    }
}
```


 3. 配置AndroidManifest.xml

#### 自定义ContentResolver