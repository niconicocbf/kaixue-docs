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
这次我们要说的东西，它并不是数据库本身，它是一个对 sqlite 数据库( Android 第一方数据库)的拓展专业一点的说法就是再封装， sqlite 原本的性能虽然很强大，但是易用性比较差，很多时候还需要我们自己用 String 拼接 sql 语句来控制数据库来创建表已经增删改查等操作比如：
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
````
单独使用Sql数据库还是比较原始的，写出来的sql语句是用String拼接的并没有语法检查，并且不会在编译阶段得到验证,容易出错。
所以目前的Android开发的大环境下，一些开发的老手都倾向于使用第三方的比较易用且稳定的数据库，比较著名的有realm，GreenDao，LitePal等，这些数据库都一个共通点：他们都提供ORM(Object-relational mapping)功能。
问题来了，什么是ORM()？
我们先看一个例子。

我们有一个book类，我们有一个数据库library，现在面对的需求是:我们想要遍历整个数据库找到library里面author为“Linus”的书，并直接拿到book_list的列表对象.如果我们用非ORM的库写会是这样的
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
````
我们手写sql语句后拿到了想要的数据集对象data,但是data本身并不是Book对象的列表,后面我们还需要遍历data,拿到data中每一个对象逐个转化成book对象并添加到book_list中.

如果我们使用ORM库,代码是这样的:
```java
book_list = new List();

book_list = BookTable.query(author="Linus");
```
所以ORM的主要职责是帮助开发者自动完成数据库对象和实体类之间的相互转换.当然,部分ORM数据库还顺便封装了大部分sql语句,让数据库数据检索变得更加简洁,方便.这样的好处显而易见:
- 你不再需要写生硬的sql语句(部分ORM库已经封装了绝大部分的sql语句), Room 中的Sql语句会在编译阶段得到验证.
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

   ````java
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

  public class Address {
      public String street;
      public String nation;
      public String posetCode;
  }

       ````

    这里,本文对注解的意义进行简单的讲解以帮助开发者更好的上手Room.

    ##### @Entity & @PrimaryKey :
    在上述例子中使用了 @Entity(tableName = "user") 表明这是一张数据库表,通过参数 tableName 可以改变表的名字.当然你也可以不添加 tableName 参数定义表名 .Room会默认使用类名 User 作为表名. @Entity 除了 tableName 以外还有许多其他属性,比如: primaryKeys=“***” 这样可以不通过 @PrimaryKey 指定主键.
    当我们想使用可以自增的主键时,可以设置 @PrimaryKey 的 autoGenerate 属性.具体用例参考下面的代码块.

    此外还可以通过 @Entity 来建立索引,参考更多[索引](https://zhuanlan.zhihu.com/p/23624390)的相关内容.为确保索引中某个字段或者多个字段形成的组唯一,我们可以使用 unique = true 来防止 name 和 telephoneNumber 同时拥有相同的属性.( unique 默认为false的,以下的代码块为例,若 unique = false 这意味着数据库准许同时存储 User 对象 A , B .其中A.MingZi==B.MingZi && A.telephoneNumber==B.telephoneNumber.反之如果 unique=true 你试图同时存储 A , B 会引起报错.)

    ````java
    @Entity(tableName = "user",indices = @Index(value = {"name","userId"},unique = true))
    public class User {
        ...
        @ColumnInfo(name = "name")
        private String MingZi;

        private String telephoneNumber;

        @PrimaryKey(autoGenerate = true)
        @NonNull
        private String userId;
        ...
      }
    ````



  ###### 通过 @Entity 注解的 foreignKeys 来指定外键约束和指定外键行为:

  user:

  | name     | userId        |
  | -------- |:-------------:|
  | userJohn | 66            |
  | userLuo  | 11            |
  | userhary | 22            |
  repo:

  | repoUrl     | uid           | repoId |
  | ----------- |:-------------:| :-----:|
  | userJohn.io | 66            |  1     |
  | userLuo.io  | 11            |  2     |
  | userhary.io | 22            |  3     |


  在类 **GitRepo** 中,通过 **@Entity** 注解创建了新的数据库表 **“ repo ”** 并指定了 该外键的关联对象为通过 **User.class** 类所创建的表格.下面代码指定了 **GitRepo.class** 中的 **uid** 与 **User.class** 中的 **userId** 进行约束绑定. **onDelete = CASCADE** 则指定了约束绑定后的行为,添加了该属性后:
  若 下面表格 **“user”** 中某一条数据 **userJohn** 被删除,会引起表格 ”repo” 中的 **uid == userJohn.userId** 所有对象被删除.这里 **userJohn.io** 所在的那一行会被自动出数据库中除去这是一种数据绑定的关系. **user** 表格中的变化会引起 **repo** 表格中相对应的一些数据产生变化.

  基于本文的意图上手,关于 **Room** 的外键约束,这里不做过多的描述.这里给读者一些启发 **@Entity** 中不仅有 **onDelete** 可以指定删除时的外键约束行为 还可以指定 **onUpdate** 的外键约束行为,并且 约束行为也分为 **5** 种 **NO_ACTION, RESTRICT, SET_NULL, SET_DEFAULT, CASCADE** . 在不特意指定外键行为的情况下 **onDelete** 和 **onUpdate** 的值默认为 **NO_ACTION** ,这种情况下如果直接使用 **DAO** 中的方法删除 **user** 表中的 **userJohn** 那一行,由于 **repo** 表中 **uid** 和 **user** 表中的 **userId** 的值是一致的,此次删除会被驳回(导致报错 关联 **Exception:NOT NULL constraint failed: repo.uid**).



```java

    User.class:

    @Entity(tableName = "user",indices = @Index(value = {"name","telephoneNumber"},unique = false))
    public class User {
                        ...非关键代码省略
         @ColumnInfo(name = "name")
         private String MingZi;
         @PrimaryKey
         @NonNull
         private int userId;
                        ...非关键代码省略
                      }

--------------------------------文件分割线------------------------------

     GitRepo.class:

     @Entity(tableName = "repo",foreignKeys = @ForeignKey(entity = User.class,parentColumns = "userId",childColumns = "uid",onDelete = CASCADE))
     public class GitRepo {
        @PrimaryKey(autoGenerate = true)
        public int repoId;
        public String repoUrl;
        public int uid;
        public GitRepo(){};
                        ...非关键代码省略
                  }


```



          注意:
          - 被 @Entity 标记的注解的类对构造方法有特殊要求,推荐无脑添加一个无参数的构造方法,详情可以参考完整的 class User 代码中关于关于构造方法的注释.
          - 使用了 @Entity 注解来标记一个类来创建数据库表时,一定要为这表格指定主键(未指定 PrimaryKey 时 android studio 在编译时将会报错: error: An entity must have at least 1 field annotated with @PrimaryKey ),这里我们通过 @PrimaryKey 来指定主键,被指定为主键的成员变量必须加上 @NonNull 注解.
          - 为了保存字段，这个字段对应的成员变量需要有可以访问的gettter/setter方法或者它本身public的.
          - 使用 autoGenerate = true 属性的主键成员变量必须为数字类型( long , int ).

  ##### @Ignore :
    如果数据类有一些成员变量,你不想将其存入数据库,你希望Room能忽略该字段.则可以使用 @Ignore 注解.

  ##### @ColumnInfo :
    默认的情况下(没有指定@ColumnInfo(name = "***")的情况下) Room 使用变量名作为数据库字段名,通过 @ColumnInfo 注解开发者可以更加灵活的操作数据库字段名.
  ##### @Embedded
    以上的一些标记只适用于类的基本类型的数据固定,如果你想存储类中类,可以使用 @Embedded 注解,Room 会自动帮你完成后台的数据库字段和类之间的转化工作.
  注意:下面代码例中 Address 并没有添加 @Entity 注解, Room 不会为没添加 @Entity 的类创建一张表,实际上只会原封不动的存储 Address 中的字段(street,nation...).也就是说实际上 Room(Sqlite) 数据库是不支持实体嵌套的.


   ````java
  @Entity
  public class User {
      ...
      @Embedded
      private Address address;
      ...
  }

  public class Address {
        public String street;
        public String nation;
        public String posetCode;
    }
   ````
   ##### @Embedded

   ### 创建 DAO (data access object)

   创建 **Room** 的数据库Dao,需要创建一个 **interface** 并给它加上 **@Dao** 注解.我们在该接口中定义操作数据的方法,为这些方法加上相应的 Room 注解,通过注解的属性控制增删改查的具体内容,在编译阶段, Room 会根据注解和 Dao 内方法的返回值和参数为我们生成具体 Dao 的实现类,同时会对 Dao 内中的 Sql 进行语法检查,确保数据库操作方法的有效性.

   下面是一个简单的DAO（接口），定义了本指南的所有CRUD操作。

   ````java
@Dao
public interface UserDAO {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void Userinsert(User... users);

    @Delete
    public void Userdedlete(User... users);

    @Delete
    public void UserdeleteWithList(List<User> users);

    @Update(onConflict = OnConflictStrategy.REPLACE)
    public void Userupdate(User... users);

    @Query("SELECT * FROM user WHERE Name= :name")
    public User getUserWithName(String name);

    @Query("SELECT * FROM user")
    public List<User> getAllUser();

    @Insert
    public void Repoinsert(GitRepo... repos);

    @Delete
    public void repoDelect(GitRepo... repos);

    @Query("SELECT * FROM repo")
    public List<GitRepo> getAllRepo();
}

   ````
  ##### @Insert
  上述代码中,为 Userinsert 方法加上了 @Insert 注解,同时给注解设置了 onConflict 属性来设定进行数据库操作时遇到冲突的解决办法.同时给该方法加上了 (User... users) 参数表明该方法可以传入一个或者多个 User 的对象.该注释本身的使用比较简单也易于理解这里,我主要对 onConflict 属性的的参数进行详细讲解.查看源码可知 onConflict 一共有五种参数如下图所示: REPLACE , ROLLBACK ....
  ````java
  @Retention(SOURCE)
  @IntDef({OnConflictStrategy.REPLACE, OnConflictStrategy.ROLLBACK, OnConflictStrategy.ABORT,
        OnConflictStrategy.FAIL, OnConflictStrategy.IGNORE})
  ````
  再仔细翻阅源码,这五个参数只有: REPLACE , ABORT , IGNORE 没有被 Deprecated 掉,这里我通过下面数据库插入方法伪代码讲解没有被 Deprecated 掉的参数的用途.
  ````java
    .....
    UserDAO userDAO = userDatabase.getUserDAO();

    /*
      User 的第二个参数为主键,这里 new 了两个主键都为 66 的对象 testUser , testUser2 .
      */
    testUser = new User("byfordd",66,"0000000","male");
    testUser2 = new User("byfordd66",66, "000000066", "male");

    userDAO.Userinsert(testUser,testUser2);

  ````

  插入数据时,如果发现表中已经有此行数据或者发生冲突（判断是否有主键相同的数据或者唯一索引判断(上文中建立了索引且 unique = true 为true的情况下)）,我们这里试图插入两个主键相同的对象来创造冲突.
  - OnConflictStrategy.REPLACE

插入数据时,出现冲突,则先删除冲突的数据，然后插入新的数据。否则，直接插入新数据。也就是最终数据库里只会剩下 testUser2.

  - OnConflictStrategy.IGNORE

插入数据时,出现冲突,直接跳过出现冲突的语句.也就是说最终 testUser2 的插入操作被无效化了,此时不会报错, App 也不会崩溃.

  - OnConflictStrategy.ABORT

该值为 onConflict 的默认属性,插入数据时,出现冲突,直接报错, App 崩溃,关联 Exception : SQLiteConstraintException: UNIQUE constraint failed: user.userId.

##### @Delete
被该注解标注的方法可以通过传入一个或多个参数来对数据库对应的数据进行删除,也可以传入 List 来一次性删除多个对象.
在这里,我认为要主要谈谈 Sqlite 数据库中数据的身份认定的问题, 实验发现使用 Room 的 @Delete 注解标注的方法删除数据时只会参照主键的值来判断对象在数据库是否有对应数据.也就是说主键的值是数据身份的唯一标实.如下图伪代码所示:代码中创建了 testUser , testUser2 两个 User 对象,然后使用了 dao 对象的方法将 testUser 的数据插入数据库中,又使用了 testUser2 对象将 testUser 的数据从数据库中删除.
````java
testUser = new User("byfordd", 66, "0000000", "male");
testUser2 = new User("byfordd66", 66, "000000066", "male66");
userDAO.Userinsert(testUser);
userDAO.Userdedlete(testUser2);

````

##### @Update
  在讲解 @Delete 注解中,我们知道 ***主键*** 是数据库中数据的唯一标实.在使用被 @Update 标注的方法 Userupdate(User... users) 更新数据时,数据库会检测数据库中是否有和被传入数据主键值相同的数据,一旦检测到了,就会修改这条数据为传入数据的值,如果没有检测到则什么都不做.

  另外 @Update 不能用于更新主键的值,若想要更改主键的值,必须通过先删除这条数据再插入的方法.

  该注解和 @Insert 类似,也可以添加 onConflict 属性来指定遇到数据库更新冲突时的处理办法

  为了创造数据库 update 冲突的情况,下面例子中 我创建了一个 name 和 telephoneNumber 的索引,并设置它的 unique 为 true.这样我们在操作数据时,如果试图使数据库中出现 name 和 telephoneNumber 完全相同的数据时,就会产生冲突.

  ````java
  @Entity(tableName = "user",indices = @Index(value =   {"name","telephoneNumber"},unique = true))
  public class User {
              ...
              public User(String name, int userId, String telephoneNumber) {
              name = name;
              this.userId = userId;
              this.telephoneNumber = telephoneNumber;
              ...
              }
            }
  --------------------------------------------------
        // new User(String name,int userId,String telephoneNumber);
        testUser = new User("byfordd", 22, "0000000");
        testUser1 = new User("byfordd22", 33, "000000022");
        testUser2 = new User("byfordd22", 22, "000000022");
        userDAO.Userinsert(testUser);
        userDAO.Userinsert(testUser1);
        userDAO.Userupdate(testUser2);
  ````
  - OnConflictStrategy.REPLACE

如上述代码,修改数据时,执行到 userDAO.Userupdate(testUser2); 出现冲突,该例中 testUser2 会替换掉数据库中的 testUser1 的数据,也就是最后数据库只会剩下 testUser2 的数据.

  - OnConflictStrategy.IGNORE

修改数据时,出现冲突,直接跳过出现冲突的语句.也就是说最终 testUser 的修改操作被无效化了,此时不会报错, App 也不会崩溃.本例中数据库中最终还剩下  testUser 和 testUser1 的数据.

  - OnConflictStrategy.ABORT

该值为 onConflict 的默认属性,修改数据时,出现冲突,直接报错, App 可能崩溃.

##### @Query
@Query 是 Room Dao 中最重要的注解,它可以用于数据库的读写操作.每一个 @Query 里的 sql 方法都会在编译的阶段被验证.所以不会产生 runtime failure .
