---
title: SQLite数据库的使用之运用SQL语句
date: 2016-8-30 13:00:25
tags: [数据库,Android]
---

# SQLite数据库的使用之运用SQL语句

​        数据库对于仍和一个数据量较大的程序来讲都是非常重要。这是一个非常轻量级的数据库，对于每个程序来讲，都有一个私密的数据库，别的程序看不到，必需要用内容提供器才能给别的app使用，所以说是十分安全的，SQLite没有用户。今天学习了用execSQL来进行SQlite的插入查询。

<!-- more -->

------

## SQLite的创建

SQLite的创建非常简单，使用一个SQLiteDatebase类即可

```
 SQLiteDatabase db = openOrCreateDatabase("temp.db", MODE_PRIVATE, null);
```

SQlite同样不能直接获取实例，需要用openOrCreateDatabase来获得，在没有创建数据库时创建，在有数据库时打开，比较万能。第一个参数很明显时数据库名，最好以.db来结尾。第二个参数是创建方式，推荐使用MODE_PRIVATE，第三个参数暂时以null结尾。

------

## 使用SQL语句操作数据库

SQL语句一般直接保存在String中当作参数，传入database的execSQL方法的参数中，代码如下：

```
db.execSQL("create table if not exists usertb (_id integer primary key autoincrement," +
                "name text not null," +
                "age integer not null," +
                "sex text not null" +
                ")");
        db.execSQL("insert into usertb(name,age,sex) values('张三',18,'男')");
        db.execSQL("insert into usertb(name,age,sex) values('李四',20,'男')");
        db.execSQL("insert into usertb(name,age,sex) values('王五',19,'男')");
```

------

## 使用Cursor来查询数据库

数据库创建完成了，自然要轮到查询工作了，使用

```
Cursor cursor=db.rawQuery("select * from usertb",null);
```

这里的 `select * from usertb` 表示从表为usertb中获得所有行。rawQuery会返回一个Cursor对象，只是一个游标，默认指向第一行数据的上方，查询代码：

```
 if (cursor != null) {
            while (cursor.moveToNext()) {//cursor默认从第一条之上开始查询
                Toast.makeText(MainActivity.this, "正在查询"+cursor.getString(cursor.getColumnIndex("name")), Toast.LENGTH_SHORT).show();

            }
            cursor.close();
        }
```

这句代码`cursor.getColumnIndex("name")` 表示从行中要提取名为name的值，最后最好使用`cursor.close()` 将cursor对象关闭，否则当查询数据过多时，会发生内存报错。

------

## 关闭数据库

最后的一部当然也别忘了`db.close();` 来将数据库关闭。