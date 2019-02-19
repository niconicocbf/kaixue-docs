它是什么？
它解决了什么问题？
它怎么用？(这里其实就是组件的基本操作 如 Room基本的CRUD)
它在真实开发过程中使用的案例是什么样的？
它内部是怎么去做的？(这个可以放在文章最末出去讲解,但是不要大段源码贴上来,讲出最关键的几个就好,主要是思想！)
总结,付源码地址(开源地址)

注解讲解
数据库更新
多表联查
和其他组件配合使用, Rxjava LiveData

https://stackoverflow.com/questions/1279613/what-is-an-orm-and-where-can-i-learn-more-about-it

# 写给Android开发者的Room简易上手指南
相信大多数Android开发者，初学Android开发的时候，使用过Sqlite(关系型数据库)来储存，持久化数据。而Room library为我们的老朋友带来了一次最大的升级。
## Room的全称叫 Room Persistence Library
这次我们要说的东西，它并不是数据库本身，它是一个对sqlite数据库(Android第一方数据库)的拓展，sqlite原本的性能虽然很强大，但是易用性比较差，很多时候还需要我们自己用String拼接sql语句来控制数据库来创建表已经增删改查等操作比如：
````
```java
DatabaseHelper.java
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;
import java.util.List;
public class DatabaseHelper extends SQLiteOpenHelper {

  //创建数据库表用都SQl语句
  public static final String CREATE_TABLE =
            "CREATE TABLE " + "TABLE_NAME" + "("
                    + "COLUMN_ID" + " INTEGER PRIMARY KEY AUTOINCREMENT,"
                    + "COLUMN_NOTE" + " TEXT,"
                    + "COLUMN_TIMESTAMP" + " DATETIME DEFAULT CURRENT_TIMESTAMP"
                    + ")";

    // Database Version
    private static final int DATABASE_VERSION = 1;

    // Database Name
    private static final String DATABASE_NAME = "my_db";


    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    // Creating Tables
    @Override
    public void onCreate(SQLiteDatabase db) {

        // create notes table
        db.execSQL(CREATE_TABLE);
    }

}
```
````
单独使用Sql数据库还是比较原始的，自己写的sql语句并没有语法检查，容易出错。
所以目前的Android开发的大环境下，一些开发的老手都倾向于使用第三方的比较易用且稳定的数据库，比较著名的有realm，GreenDao，LitePal等，这些数据库都一个共通点：他们都提供ORM(Object-relational mapping)功能。
问题来了，什么是ORM()？
我们先看一个例子。

我们有一个book类，我们有一个数据库library，现在面对的需求是:我们想要遍历整个数据库找到library里面author为“Linus”的书，并直接拿到book_list的列表对象.如果我们用非ORM的库写会是这样的
````
```java
book_list = new List();
sql = "SELECT book FROM library WHERE author = 'Linus'";
data = query(sql); // I over simplify ...
while (row = data.next())
{
     book = new Book();
     book.setAuthor(row.get('author');
     book_list.add(book);
}
```
````
我们手写sql语句后拿到了想要的数据集对象data,但是data本身并不是Book对象的列表,后面我们还需要遍历data,拿到data中每一个对象逐个转化成book对象并添加到book_list中.

如果我们使用ORM库,代码是这样的:
````
```java
book_list = new List();
book_list = BookTable.query(author="Linus");
```
````
所以ORM的主要职责是帮助开发者自动完成数据库对象和实体类之间的相互转换.当然,部分ORM数据库还顺便封装了大部分sql语句,让数据库数据检索变得更加简洁,方便.这样的好处显而易见:
- 你不再需要写生硬的sql代码(部分ORM库已经封装了绝大部分的sql语句)
- 一些模版代码,例如类的类型转换,ORM库已经帮你自动完成
- 代码更加简洁,易读,可以有更多精力的关注业务逻辑.
## 为什么要用Room
笔者认为,Android端的数据库在性能上其实差距并不明显,不同的数据库在处理不同数量的数据时,各有优势.而且数据库在app中是属于运行效率特别高的一环,数据库的性能在android设备中不构成性能瓶颈,所以在这里并不讨论性能.当然,用过sqlite的开发者都说sqlite的性能很不错.
一般来说我会在一下几种情况推荐使用Room:
- 开发时希望数据库简单易用,写出来的代码逻辑清晰,简洁易读,便于维护.
- Room库属于Jetpack中的一个子库,它既可以独立运行,完成数据库的职责,还可以和各种其他库完成各种奇妙的Combo,例如Room+ViewModel+LiveData可以快速帮开发者完成数据库数据和UI间的数据同步.
- 谷歌亲儿子,第一方最好用的数据库,在安全性上有保证,并且后续还会得到各种更新升级.不仅有JetPack套件的支持,第三方的库例如Rxjava等也支持Room.
