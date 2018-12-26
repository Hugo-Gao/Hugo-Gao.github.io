---
title: SQLite数据库的使用之使用内置函数
date: 2016-08-31 13:01:04
tags: [Android,数据库]
---



光会使用SQL语句来操作数据库是完全不够的，虽然看起来非常的极客范，但是你想想，万一少打一个空格，那你的程序就直接GG了，所以接下来学习的使用SQLitedatabase类的内置函数就非常关键了。

----------


## 数据库创建的##
数据库的创建和之前还是一样的，这个没法用内置函数来创建。

```
 SQLiteDatabase db = openOrCreateDatabase("stu.db", MODE_PRIVATE, null);
        db.execSQL("create table if not exists stutb(_id integer primary key autoincrement," +
                "name text not null," +
                "sex text not null," +
                "age integer not null)");
```

----------
## 使用ContentValues操作数据##
在查看内置函数的文档时，你会发现每个函数的参数都有一个ContentValues，这是什么呢？

```
This class is used to store a set of values that the ContentResolver can process.
```
ContentValues的文档里这么写到，意思是只有用这个类来储存一组值时，ContentResolver才能处理。目前我把它看作一个Hashmap，以键值对的形式存储，可以直接new出对象。

```
 ContentValues values = new ContentValues();//暂时理解为一个hashmap
        values.put("name","张三");
        values.put("sex","男");
        values.put("age",18);
```

----------
## 插入数据库 ##
插入数据库直接用`long rowId=db.insert(TABLENAME, null, values);` 第一个参数不用说，第二个参数不用管，第三个参数就是我们刚刚创建的ContentValues。它会返回一个long型的id。
**注意的是values使用过后如果还想继续插入，必须要`values.clear()`** 

那么我们再多插入几个数据

```
values.clear();//记得清空
        values.put("name","李四");
        values.put("sex","男");
        values.put("age",29);
        db.insert(TABLENAME, null, values);

        values.clear();//记得清空
        values.put("name","李无");
        values.put("sex","男");
        values.put("age",39);
        db.insert(TABLENAME, null, values);

        values.clear();//记得清空
        values.put("name","李撒旦");
        values.put("sex","男");
        values.put("age",39);
        db.insert(TABLENAME, null, values);

        values.clear();//记得清空
        values.put("name","李打撒");
        values.put("sex","男");
        values.put("age",99);
        db.insert(TABLENAME, null, values);

        values.clear();
```
这样我们的数据库就有5条数据了

----------
## 更新与删除##
那么想在如果我想把所有id>3的数据的性别全部改为女怎么办呢?
使用内置函数的update方法，并且我们刚刚说到使用内置函数操作数据库必定绕不开ContentValues。

```
 values.put("sex","女");
 db.update(TABLENAME, values, "_id>?", new String[]{"3"});
```
在values中放置我们想要修改的键值，传入update函数，第三个参数用“？”表示占位，具体的值在第四个参数上填上，第四个参数必须是字符串数组。
那么接下来我想要删除名字里有"打"的数据，毫无疑问调用delete函数

```
db.delete(TABLENAME, "name like ?", new String[]{"%打%"});
```
用法与更新差不多

----------


## 查找操作 ##
查找操作稍微繁琐一点，调用query函数返回Cursor对象

```
        Cursor cursor=db.query(TABLENAME, null, "_id > ?", new String[]{"0"},null,null,"name");//最后一个排序

```

```
// 调用SQLiteDatabase对象的query方法进行查询，返回一个Cursor对象：由数据库查询返回的结果集对象  
            // 第一个参数String：表名  
            // 第二个参数String[]:要查询的列名  
            // 第三个参数String：查询条件  
            // 第四个参数String[]：查询条件的参数  
            // 第五个参数String:对查询的结果进行分组  
            // 第六个参数String：对分组的结果进行限制  
            // 第七个参数String：对查询的结果进行排序  
```

参数看起来很多吧！其实我们就用到其中的几个参数而已，第三个第四个参数组成查找条件，这里我们查找所有的数据，最后一个参数为“orderBy”即排序方法，我们按名字排序。
返回一个cursor对象我们这里还是先检查是否为空

```
if (cursor != null) {
            String[] columns = cursor.getColumnNames();
            while (cursor.moveToNext()) {
                for (String name : columns
                        ) {
                    Log.d("info", name+"为");
                    Log.d("info", " 查询到"+cursor.getString(cursor.getColumnIndex(name)));

                }
            }
            cursor.close();
        }
```
接着我们用一个字符串数组把所有列的名字全部保存起来，使用`cursor.getColumnNames();` 即可得到所有列的名字，接下来就移动游标，游标指定了行，使用`getColumnIndex` 来确定该列的数据。

----------


最后要记得关闭我们的游标和数据库。

----------
## SQLiteOpenHelper的使用 ##
SQLiteOpenHelper是一个帮助类，是一个抽象类，因此必须要自己新建一个类去继承他，需要重写里面的方法

```
 @Override //首次建库是使用，可以把建库建表操作放在里面
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("create table if not exists studb(_id integer primary key autoincrement ," +
                "name text not null," +
                "sex text not null," +
                "age integer not null)");
        db.execSQL("insert into studb(name,age,sex) values('张三',18,'男')");
    }
```

```
 @Override//当数据库的版本发生变化的时候 会自动执行
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
```
新建好这个类后，我们就可以在其他类中使用了，先new 一个helper对象，传入当前上下文以及数据库名，接着调用getWritableDatabase方法就会返回一个数据库，那么接下来的操作就和以前一样了。

```
 DBOpenHelper helper = new DBOpenHelper(MainActivity.this, "stu.db");
        SQLiteDatabase db = helper.getWritableDatabase();//read 只读
        Cursor cursor = db.rawQuery("select * from studb",null);
        if (cursor != null) {
            String[] cols = cursor.getColumnNames();
            while (cursor.moveToNext()) {
                for (String name:cols
                     ) {
                    Log.d("info", cursor.getString(cursor.getColumnIndex(name)));
                }
            }
            cursor.close();
        }
        db.close();
```

