### 概述
　GreenDAO号称是最快的ORM框架，能够提供一个接口通过操作对象的方式去操作关系型数据库，它能够让你操作数据库时更简单、更方便

> 优点

 1. 性能高，号称Android最快的关系型数据库
 2. 内存占用小
 3. 库文件比较小，小于100K，编译时间低，而且可以避免65K方法限制
 4. 支持数据库加密  greendao支持SQLCipher进行数据库加密 有关SQLCipher可以参考这篇博客Android数据存储之Sqlite采用SQLCipher数据库加密实战
 5. 简洁易用的API

> GreenDao3.0的改动
 
使用过GreenDao的同学都知道，3.0之前需要通过新建GreenDaoGenerator工程生成Java数据对象（实体）和DAO对象，非常的繁琐而且也加大了使用成本，
GreenDao  3.0最大的变化就是采用注解的方式通过编译方式生成Java数据对象和DAO对象

