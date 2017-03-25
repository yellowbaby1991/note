1. [简介](#简介)
2. [配置方法](#配置方法)
3. [创建数据库](#创建数据库)
4. [更新数据库](#更新数据库)
5. [添加数据](#添加数据)
6. [更新数据](#更新数据)
7. [删除数据](#删除数据)
8. [查询数据](#查询数据)

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

 1. 传统方式想更新数据库如何保存数据是一个大问题，但是LitePal帮我们统统都解决了，想添加字段，就在bean对象中添加一个新属性，想添加表格就添加一个新的bean
 
``` java
package yellow.com.litepal;

import org.litepal.crud.DataSupport;

public class Book extends DataSupport {

    ...
    private String press;

    public String getPress() {
        return press;
    }

    public void setPress(String press) {
        this.press = press;
    }
	...
}
```


 2. 当然记得同时更新litepal.xml，并且将版本号+1

``` xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>
    <dbname value="BookStore" />
    <version value="2" />
    <list>
        <mapping class="yellow.com.litepal.Book"></mapping>
        <mapping class="yellow.com.litepal.Category"></mapping>
    </list>
</litepal>
```



















 [1]: https://github.com/LitePalFramework/LitePal
#### 添加数据

 1. 让Bean类继承DataSupport类

``` java
public class Book extends DataSupport {
	...
}
```
 2. 创建Bean对象后调用save方法

``` java
Book book = new Book();
book.setName("The Da Vinci Code");
book.setAuthor("Dan Brown");
book.setPages(454);
book.setPrice(16.96);
book.setPress("Unknow");
book.save();
```

#### 更新数据

 1. 常规修改：设置需要修改的值，调用updateAll添加查询条件

``` java
/**
 * 等价于 update table set price = 14.95,press = 'Anchor' where name = 'The Lost Symbol' and author = 'Dan Brown'
 */
Book book = new Book();
//设置需要修改的值
book.setPrice(14.95);
book.setPress("Anchor");
//设置查询条件
book.updateAll("name = ? and author = ?", "The Lost Symbol", "Dan Brown");
```

 2. 将某个属性设置为默认值，需要调用setToDefault方法

``` java
Book book = new Book();
book.setToDefault("press");
//设置查询条件
book.updateAll("name = ? and author = ?", "The Lost Symbol", "Dan Brown");
```


#### 删除数据

 1. 调用过save方法或者使用查询API得到的对象可以直接调用delete()方法删除
 2. 和updateAll类似，deleteAll删除

``` java
DataSupport.deleteAll(Book.class, "price < ?", "15");
```

#### 查询数据

 1. 使用DataSupport.select多条件查询
 
``` java
//查询Book表中第11-20页并且满足页数大于400这个条件的name，author，pages这3列数据
 List<Book> books = DataSupport.
 select("name", "author", "pages").
 where("pages > ?", "400").
 order("pages").
 limit(0).
 offset(10).
 find(Book.class);
```

 2. 支持原生的SQL查询得到Cursor

``` java
Cuscor c = DataSupport.findBySQL("select * from Book where pages >? and price < ?","400","20"); 
```


