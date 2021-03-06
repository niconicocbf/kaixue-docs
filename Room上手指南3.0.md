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
单独使用Sql数据库还是比较原始的，写出来的sql语句是用String拼接的并没有语法检查，并且不会在编译阶段得到验证,容易出错。
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
- 你不再需要写生硬的sql语句(部分ORM库已经封装了绝大部分的sql语句),Room中的Sql语句会在编译阶段得到验证.
- 一些模版代码,例如类的类型转换,ORM库已经帮你自动完成.
- 代码更加简洁,易读,可以有更多精力的关注业务逻辑.
## 为什么要用Room
笔者认为,Android端的数据库在性能上其实差距并不明显,不同的数据库在处理不同数量的数据时,各有优势.而且数据库在app中是属于运行效率特别高的一环,数据库的性能在android设备中不构成性能瓶颈,所以在这里并不讨论性能.当然,用过sqlite的开发者都说sqlite的性能很不错.
一般来说我会在一下几种情况推荐使用Room:
- 开发时希望数据库简单易用,写出来的代码逻辑清晰,简洁易读,便于维护.
- Room库属于Jetpack中的一个子库,它既可以独立运行,完成数据库的职责,还可以和各种其他库完成各种奇妙的Combo简化代码逻辑,例如Room+ViewModel+LiveData可以快速帮开发者完成数据库数据和UI间的数据同步快速构建MVVm模型.
- 谷歌亲儿子,第一方最好用的数据库,在安全性上有保证,并且后续还会得到各种更新升级.不仅有JetPack套件的支持,第三方的库例如Rxjava等也支持Room拓展.
 ## 开始使用Room时你需要知道的一切

    #### 项目配置
    创建一个新的 Android Studio Project:HelloRoom
    在 App 层的 build.gradle 文件中添加下面依赖:
    ````
    ```Groovy
    def room_version = "2.1.0-alpha04"

    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version" // For Kotlin use kapt instead of annotationProcessor

    ```
    ````
    Tips:
    鉴于很多开发者对JetPack的依赖添加还存有疑虑,这里提供两个非常有用的网址,供大家参考:
    [JetPack各组件的最新版本以及过去版本号](https://dl.google.com/dl/android/maven2/index.html),
    [使用Room时根据情况需要的不同依赖的参考](https://developer.android.com/jetpack/androidx/releases/room)
    确保 project 层的 build.gradle 中的 repositories 添加了google()
    ````
    ```Groovy
    repositories {
        google()
        jcenter()
      }
    ```
    ````
    #### 建表
    使用Room数据库时,你不需要特意写一个Class类去继承 SQLiteOpenHelper ,或者写建表语句.只要我们在对应的数据类上使用 @Entity 注解.Room会自动帮你创建这些数据库表.这里,我创建了一个数据类 User 并给它加上了相应的注解.

   ````
   /*  也可以通过 @Entity 注解在指定 tableName 的同时,指定 primaryKeys .
    例: @Entity(tableName = "user",primaryKeys = "userId")
    */
@Entity(tableName = "user")
public class User {
    @ColumnInfo(name = "name")
    private String MingZi;

    /* @PrimaryKey ,被标注的成员变量为该数据表的主键.在一个类中(同一张表中)
    ,主键有而且只能有一个.
    @NonNull 必须
    */
    @PrimaryKey
    @NonNull
    private String userId;

    private String telephoneNumber;

    private String sex;

    @Embedded
    private Address address;

    private int age;

    @Ignore
    private String ignoreMsg;

    /*
    * 当存在有参数的构造方法时,必须再添加一个空的构造方法或者添加一个不含被 @Ignore 标记的成员变量的构造方法,推荐使用空参构造方法
    * */
    public User() {}

    public User(String mingZi, @NonNull String userId, String telephoneNumber, String sex, Address address, int age, String ignoreMsg) {
        MingZi = mingZi;
        this.userId = userId;
        this.telephoneNumber = telephoneNumber;
        this.sex = sex;
        this.address = address;
        this.age = age;
        this.ignoreMsg = ignoreMsg;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    public Address getAddress() {
        return address;
    }

    public String getMingZi() {
        return MingZi;
    }

    public void setMingZi(String mingZi) {
        MingZi = mingZi;
    }

    @NonNull
    public String getUserId() {
        return userId;
    }

    public void setUserId(@NonNull String userId) {
        this.userId = userId;
    }

    public String getIgnoreMsg() {
        return ignoreMsg;
    }

    public void setIgnoreMsg(String ignoreMsg) {
        this.ignoreMsg = ignoreMsg;
    }

    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return MingZi;
    }

    public void setName(String name) {
        this.MingZi = name;
    }
    @NonNull
    public String getTelephoneNumber() {
        return telephoneNumber;
    }
    public void setTelephoneNumber(@NonNull String telephoneNumber) {
        this.telephoneNumber = telephoneNumber;
    }
}

       ````

    这里,本文对注解的意义进行简单的讲解以帮助开发者更好的上手Room.

    @Entity & @PrimaryKey : 在上述例子中使用了 @Entity(tableName = "user") 表明这是一张数据库表,通过参数 tableName 可以改变表的名字.当然你也可以不添加 tableName 参数定义表名 .Room会默认使用类名 User 作为表名. @Entity 除了 tableName 以外还有许多其他属性,比如: primaryKeys=“***” 这样可以不通过 @PrimaryKey 指定主键.

    此外还可以通过 @Entity 来建立[索引](https://zhuanlan.zhihu.com/p/23624390).为确保索引中某个字段或者多个字段形成的组唯一我们使用 unique = true 来防止 name 和 telephoneNumber 同时拥有相同的属性.
    ````
    @Entity(tableName = "user",indices = @Index(value = {"name","userId"},unique = true))
    public class User {
        ...
        @ColumnInfo(name = "name")
        private String MingZi;

        @PrimaryKey
        @NonNull
        private String userId;
        ...
      }
    ````

    注意:
    1.被 @Entity 标记的注解的类对构造方法有特殊要求,详情可以参考注释上述代码的注释.

    2.使用了 @Entity 注解来标记一个类来创建数据库表时,一定要为这表格指定主键(未指定 PrimaryKey 时 android studio 在编译时将会报错: error: An entity must have at least 1 field annotated with @PrimaryKey ),这里我们通过 @PrimaryKey 来指定主键,被指定为主键的成员变量必须加上 @NonNull 注解.

    3.为了保存字段，这个字段对应的成员变量需要有可以访问的gettter/setter方法或者它本身public的.

    @Ignore :如果有一些信息,你不想将其存入数据库,你希望Room能忽略该字段.则可以使用 @Ignore 注解.

    @ColumnInfo :默认的情况下(没有指定@ColumnInfo(name = "***")的情况下) Room 使用变量名作为数据库字段名,通过 @ColumnInfo 注解开发者可以更加灵活的操作数据库字段名.
