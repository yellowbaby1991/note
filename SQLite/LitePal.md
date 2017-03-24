#### 简介
　　LitePal是一种为Android SQLlite设计的ORM框架，使用面向对象的思维来操作数据库
　　[GitHub地址][1]

#### 配置方法

 1. 在app/build.gradle中添加依赖
 
``` xml
dependencies {
    ...
    compile 'org.litepal.android:core:1.3.2'
    ...
}
```
 2. 在app/src/assets下新建一个litepal.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>
    <dbname value="BookStore" />
    <version value="1" />
    <list></list>
</litepal>
```
 3. 修改AndroidManifest.xml的application name属性

``` xml
<manifest>
    <application
        android:name="org.litepal.LitePalApplication"
        ...
    >
        ...
    </application>
</manifest>
```


 
 
 
 
 
 
 
 
 
 
 
 
 [1]: https://github.com/LitePalFramework/LitePal
 
#### 创建数据库

 1. 创建一个Bean对象，如Book.java
 
``` java
package yellow.com.litepal;

import org.litepal.crud.DataSupport;

public class Book extends DataSupport {

    private int id;

    private String author;

    private double price;

    private int pages;

    private String name;

    private String press;

    //getter && setter
}

```

 2. 在litepal.xml中添加一个mapping
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>
    <dbname value="BookStore" />
    <version value="2" />
    <list>
        <mapping class="yellow.com.litepal.Book"></mapping>
    </list>
</litepal>
```

3. 调用getDatabase方法

``` java
Button createDatabase = (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Connector.getDatabase();
            }
        });
```





#### 更新数据库

 1. List item