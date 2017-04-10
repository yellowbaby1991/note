* [内容提供器是什么](#内容提供器是什么)
* [自定义内容提供器](#自定义内容提供器)
	* [创建ContentProvider](#创建contentprovider)
	* [使用ContentResolver](#使用contentresolver)

### 内容提供器是什么
　　ContentProvider主要用于不同程序间数据共享，是IPC的一种方式，提供了一套完整的机制运行跨进程通信，并且同时还可以保证被访问数据的安全性
  
### 自定义内容提供器
#### 创建ContentProvider

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

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="app.yellow.contentprovider">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        ...

        <provider
            android:name=".MyProvider"
            android:authorities="com.yellow.cp.contacts"
            android:exported="true"></provider>
    </application>

</manifest>
```


#### 使用ContentResolver

　　客户端使用ContentResolver通过URI访问Provider的增删改查方法

``` java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private ListView mContactsLv;
    private List<String> mContactsName;
    private ArrayAdapter<String> mAdapter;
    private ContentResolver mCr;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initListView();

        mCr = getContentResolver();
    }

    private void initListView() {
        mContactsLv = (ListView) findViewById(R.id.contacts_lv);
        mContactsName = new ArrayList<>();
        mAdapter = new ArrayAdapter(this, android.R.layout.simple_list_item_1, mContactsName);
        mContactsLv.setAdapter(mAdapter);
        mContactsLv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                Uri uri = Uri.parse("content://com.yellow.cp.contacts/contacts/" + mContactsName.get(position));
                Cursor cursor = mCr.query(uri, null, null, null, null);
                while (cursor.moveToNext()) {
                    String phone = cursor.getString(cursor.getColumnIndex("phone"));
                    Toast.makeText(MainActivity.this, phone, Toast.LENGTH_SHORT).show();
                }
            }
        });
        mContactsLv.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
            @Override
            public boolean onItemLongClick(AdapterView<?> parent, View view, int position, long id) {
                Uri uri = Uri.parse("content://com.yellow.cp.contacts/contacts/" + mContactsName.get(position));
                mCr.delete(uri, null, null);
                query(view);
                return true;
            }
        });
    }

    public void insert(View view) {
        ContentValues values = new ContentValues();
        Uri uri = Uri.parse("content://com.yellow.cp.contacts/contacts");
        for (int i = 0; i < 10; i++) {
            values.put("name", "yellowbaby" + i);
            values.put("phone", i + "");
            mCr.insert(uri, values);
        }
        query(view);
    }

    public void delete(View view) {
        Uri uri = Uri.parse("content://com.yellow.cp.contacts/contacts");
        mCr.delete(uri, null, null);
        query(view);
    }

    public void update(View view) {
        Uri uri = Uri.parse("content://com.yellow.cp.contacts/contacts");
        ContentValues values = new ContentValues();
        values.put("phone","111111");
        mCr.update(uri, values, null, null);
        query(view);
    }

    public void query(View view) {
        mContactsName.clear();
        Uri uri = Uri.parse("content://com.yellow.cp.contacts/contacts");
        Cursor cursor = mCr.query(uri, null, null, null, null);
        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex("name"));
            mContactsName.add(name);
        }
        mAdapter.notifyDataSetChanged();
    }
}
```


### 系统提供的内容提供器
#### 短信提供器

 1. 同样我们想对短信进行直接操作是不允许的，只能通过SmsProvider才可以操作SmsProvider的URI为 content://mms-sms ，UriMatcher中配置了21种类型，我们这里只用到了默认的SMS_ALL--所有短信
 2. 下面是使用ContentResolver调用SmsProvider的insert和query方法
 
``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    //安卓4.4之后只有系统应用才具有插入短信的权限
    public void insertSms(View view) {
        Uri uri = Uri.parse("content://sms");
        ContentResolver cr = getContentResolver();
        ContentValues values = new ContentValues();
        values.put("address", System.currentTimeMillis());
        values.put("read", 0);
        values.put("type", 1);
        values.put("body", "今晚见");
        cr.insert(uri, values);
    }

    public void querySms(View view) {
        Uri uri = Uri.parse("content://sms");
        ContentResolver cr = getContentResolver();
        Cursor cursor = cr.query(uri, null, null, null, null);
        while (cursor.moveToNext()) {
            String phone = cursor.getString(cursor.getColumnIndex("address"));
            long datelong = cursor.getLong(cursor.getColumnIndex("date"));
            Date date = new Date(datelong);
            int readType = cursor.getInt(cursor.getColumnIndex("read"));
            String readTypeStr = (readType == 0 ? "未读" : "已读");
            int sendType = cursor.getInt(cursor.getColumnIndex("type"));
            String sendTypeStr = (sendType == 1 ? "接受" : "发送");
            String smsbody = cursor.getString(cursor.getColumnIndex("body"));
            Log.v("520it", "电话号码:" + phone + " 读取类型:" + readTypeStr
                    + " 发送类型:" + sendTypeStr + " 短信内容:" + smsbody
                    + " 短信时间:" + date);
        }
    }
}
```

#### 联系人提供器

 1. 联系人提供器由ContactsProvider提供，URI为 content://com.android.contacts
 2. 涉及三张表raw_contacts，mimetypes，data表

　　raw_contacts保存了所有创建过的联系人，一个联系人占一行
  