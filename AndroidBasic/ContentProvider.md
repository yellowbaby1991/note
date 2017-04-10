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

　联系人提供器由ContactsProvider提供，URI为 content://com.android.contacts ，主要涉及下面三张表
 - raw_contacts表：保存了所有创建过的联系人，一个联系人占一行
 - data表：保存了联系人所有其他信息，如QQ，姓名，手机号等，外键关联raw_contact_id和mimetypes_id
 - mimetypes表：保存data表每一行数据类型，是QQ号，姓名还是手机号，但是该表会在查询的时候做成视图合并在data表中

　所以查询逻辑如下：

 1. 通过content://com.android.contacts/raw_contacts 查询raw_contacts表得到所有联系人的raw_contact_id
 2. 根据raw_contact_id通过content://com.android.contacts/data 查询data表中所有关联当前raw_contact_id的data数据
 3. 根据mimetypes的值来判断data数据类型进行输出

　插入逻辑如下：
 
 1. 通过content://com.android.contacts/raw_contacts 插入raw_contacts表一行新的数据，raw_contacts_id根据当前表的count+1得到
 2. 通过content://com.android.contacts/data 依次插入姓名，QQ，号码等信息，当然需要关联raw_contacts_id

　代码如下：
 
``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void insertContact(View view) {
        ContentResolver cr = getContentResolver();
        Uri rawContactUir = Uri.parse("content://com.android.contacts/raw_contacts");
        Cursor rawContactCursor = cr.query(rawContactUir, null, null, null, null);
        int count = rawContactCursor.getCount();

        ContentValues values = new ContentValues();
        values.put("contact_id", count + 1);
        cr.insert(rawContactUir, values);

        Uri dataUri = Uri.parse("content://com.android.contacts/data");
        ContentValues nameValues = new ContentValues();
        nameValues.put("mimetype", "vnd.android.cursor.item/name");
        nameValues.put("data1", "美女小丽");
        nameValues.put("raw_contact_id", count + 1);
        cr.insert(dataUri, nameValues);

        ContentValues phoneValues = new ContentValues();
        phoneValues.put("mimetype", "vnd.android.cursor.item/phone_v2");
        phoneValues.put("data1", "10086");
        phoneValues.put("raw_contact_id", count + 1);
        cr.insert(dataUri, phoneValues);

        ContentValues imValues = new ContentValues();
        imValues.put("mimetype", "vnd.android.cursor.item/im");
        imValues.put("data1", "202023");
        imValues.put("raw_contact_id", count + 1);
        cr.insert(dataUri, imValues);

        ContentValues emailValues = new ContentValues();
        emailValues.put("mimetype", "vnd.android.cursor.item/email_v2");
        emailValues.put("data1", "202023@qq.com");
        emailValues.put("raw_contact_id", count + 1);
        cr.insert(dataUri, emailValues);

    }

    public void readContact(View view) {
        ContentResolver cr = getContentResolver();
        Uri rawContactUir = Uri.parse("content://com.android.contacts/raw_contacts");
        Cursor rawContactCursor = cr.query(rawContactUir, new String[]{"contact_id"}, null, null, null);
        while (rawContactCursor.moveToNext()) {
            int rowContactId = rawContactCursor.getInt(0);
            Uri dataUri = Uri.parse("content://com.android.contacts/data");
            Cursor dataCursor = cr.query(dataUri, new String[]{"mimetype", "data1"}, "raw_contact_id=?", new String[]{rowContactId + ""}, null);
            while (dataCursor.moveToNext()) {
                String data1 = dataCursor.getString(dataCursor.getColumnIndex("data1"));
                String mimetype = dataCursor.getString(dataCursor.getColumnIndex("mimetype"));
                if (mimetype.equals("vnd.android.cursor.item/name")) {
                    Log.v("520it", "联系人:" + data1);
                }
                if (mimetype.equals("vnd.android.cursor.item/phone_v2")) {
                    Log.v("520it", "电话号码" + data1);
                }
                if (mimetype.equals("vnd.android.cursor.item/im")) {
                    Log.v("520it", "即时通讯" + data1);
                }
                if (mimetype.equals("vnd.android.cursor.item/email_v2")) {
                    Log.v("520it", "E-mail:" + data1);
                }
            }
        }
    }
}
```
　需要的权限
 
``` xml
    <!-- 读联系人权限 -->
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <!-- 写联系人权限 -->
    <uses-permission android:name="android.permission.WRITE_CONTACTS" />
```


  
  
### 内容观察者
　内容观察者用来观察特定Uri引起的数据变化，继而做一些相应的处理，类似数据库中的触发器，注册内容观察者的方式如下

``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Uri uri = Uri.parse("content://sms");
        ContentResolver cr = getContentResolver();
        cr.registerContentObserver(uri, true, new ContentObserver(new Handler()) {
            @Override
            public void onChange(boolean selfChange) {
                Log.v("520it", "短信数据库发生改变");
            }
        });
    }
}

```

　需要权限

``` xml
    <uses-permission android:name="android.permission.READ_SMS" />
```
