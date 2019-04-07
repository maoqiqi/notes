# 安卓架构组件 Room

> 作者：March    
> 链接：[安卓架构组件 Room 持久化类库](https://github.com/maoqiqi/blog/blob/master/pages/android_room.md)    
> 博客：http://blog.csdn.net/u011810138    
> 邮箱：fengqi.mao.march@gmail.com    
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

Room提供了一个SQLite之上的抽象层，使得在充分利用SQLite功能的前提下顺畅的访问数据库。

## 简介

SQLite是Android内置的轻量级关系型数据库，但直接使用SQLite core包做数据库操作有以下劣势：

- 需要编写长且重复的代码，这会很耗时且容易出错。
- 管理SQL困难，特别对于复杂的数据库结构。

Room是在这样的背景下应运而生。Room充当现有SQLite API的抽象层。

Room 的一些特点：

- 编译时 sql 语句检查。相信大家都有过 app 跑起来，执行到 db 语句的时候 crash，检查之后发现原来是 sql 语句少了一个 ) 或者其它符号之类的经历。
Room 会在编译阶段检查你的 DAO 中的 sql 语句，如果写错了（包括 sql 语法错误跟表名、字段名等等错误），会直接编译失败并提醒你哪里不对。
- sql 查询直接关联到 Java 对象。这个应该不用详细解释了，虽然很多第三方 db 库早已经实现。
- 耗时操作主动要求异步处理。这一点还是挺值得注意的，Room 会在执行 db 操作时判断是不是在 UI 线程。
比如当你需要插入一条记录到数据库时，Room 会让你放到异步线程去做，否则会直接 crash 掉 app 来告诉你不这样做容易阻塞 UI 线程。
- 基于注解编译时自动生成代码。这个应该算是 Room 工作原理的核心所在了。
- API 设计符合 Sql 标准。方便扩展进行各种 db 操作。

SQLite API所有必需的包，参数，方法和变量都使用简单的注解Annotation来表示。相应的Annotation如下：

- @Entity：数据模型类，对应数据库的表。你必须在Database类中的entities数组中引用这些entity类。
entity中的每一个field都将被持久化到数据库，除非使用了@Ignore注解。

- @Dao：使用一个接口类来表示Dao(Data Access Object)。DAO是Room的主要组件，负责定义查询（添加或者删除等）数据库的方法。
使用@Database注解的类必须包含一个0参数的，返回类型为@Dao注解过的类的抽象方法。Room会在编译时生成这个类的实现。

- @Database：使用此注释的类提供了一系列DAOs，它也是下层的主要连接的访问点。注解的类应该是一个抽象的继承 RoomDatabase的类。
在运行时，你能获得一个实例通过调用Room.databaseBuilder()或者 Room.inMemoryDatabaseBuilder()。

- @PrimaryKey：标识属性为表的主键
- @Insert：插入到表的数据
- @Update：更新到表数据
- @Delete：删除表的数据
- @Query：执行SQL查询

## 基本使用

1.在build.gradle文件中添加gradle依赖关系

```
dependencies {
    def room_version = "1.1.0"

    implementation "android.arch.persistence.room:runtime:$room_version"
    annotationProcessor "android.arch.persistence.room:compiler:$room_version"

    // 可选 - RxJava support for Room
    implementation "android.arch.persistence.room:rxjava2:$room_version"

    // 可选 - Guava support for Room, including Optional and ListenableFuture
    implementation "android.arch.persistence.room:guava:$room_version"

    // 测试
    testImplementation "android.arch.persistence.room:testing:$room_version"
}
```

AndroidX

```
dependencies {
    def room_version = "2.0.0-alpha1"

    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"

    // 可选 - RxJava support for Room
    implementation "androidx.room:room-rxjava2:$room_version"

    // 可选 - Guava support for Room, including Optional and ListenableFuture
    implementation "androidx.room:room-guava:$room_version"

    // 测试
    testImplementation "androidx.room:room-testing:$room_version"
}
```

2.创建数据库表的数据模型类

```
@Entity(tableName = "user")
public class User {

    @PrimaryKey
    private int uid;

    @ColumnInfo(name = "first_name")
    private String firstName;

    @ColumnInfo(name = "last_name")
    private String lastName;

    public User() {
    }

    @Ignore
    public User(int uid, String firstName, String lastName) {
        this.uid = uid;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // 此处省略了Getter/Setter方法...
}
```

划重点：

- 顶部使用 @Entity(tableName = "user") 声明这是一个实体类。其中，tableName 如果不写，那么默认类名就是表名。
- 组合主键 primaryKeys = {"firstName", "lastName"}
- 定义索引 indices = @Index(value = {"name", "sex"}, unique = true)
- 定义外键 foreignKeys = {@ForeignKey(entity = ClassEntity.class,parentColumns = "id",childColumns = "class_id")}
- 属性使用 @PrimaryKey 声明这是一个主键。@PrimaryKey(autoGenerate = true) 代表自动生成，可以理解成就是 AUTOINCRESEMENT。
- 定义数据表中的字段名 @ColumnInfo(name = "first_name")
- 指示Room需要忽略的字段或方法 @Ignore

Index索引注解可选参数

```
public @interface Index {
    // 定义需要添加索引的字段
    String[] value();
    // 定义索引的名称
    String name() default "";
    // true-设置唯一键，标识value数组中的索引字段必须是唯一的，不可重复
    boolean unique() default false;
}
```

ForeignKey外键注解可选参数

```
public @interface ForeignKey {
    // 引用外键的表的实体
    Class entity();
    // 要引用的外键列
    String[] parentColumns();
    // 要关联的列
    String[] childColumns();
    // 当父类实体(关联的外键表)从数据库中删除时执行的操作
    @Action int onDelete() default NO_ACTION;
    // 当父类实体(关联的外键表)更新时执行的操作
    @Action int onUpdate() default NO_ACTION;
    // 在事务完成之前，是否应该推迟外键约束
    boolean deferred() default false;
    // 给onDelete，onUpdate定义的操作
    int NO_ACTION = 1;
    int RESTRICT = 2;
    int SET_NULL = 3;
    int SET_DEFAULT = 4;
    int CASCADE = 5;
    @IntDef({NO_ACTION, RESTRICT, SET_NULL, SET_DEFAULT, CASCADE})
    @interface Action {
    }
}
```

3.创建Dao类，并且添加CRUD对应的抽象方法

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert
    void insertAll(User... users);

    @Update
    void update(User user);

    @Delete
    void delete(User user);
}
```

创建Dao接口类,这个接口不需要写实现类，它的实现类会由 room 根据方法上的注解自动生成。

@insert， @Update都可以执行事务操作，定义在OnConflictStrategy注解类中。

```
public @interface Insert {
    // 定义处理冲突的操作
    @OnConflictStrategy
    int onConflict() default OnConflictStrategy.ABORT;
}
```

```
public @interface OnConflictStrategy {
    // 策略冲突就替换旧数据
    int REPLACE = 1;
    // 策略冲突就回滚事务
    int ROLLBACK = 2;
    // 策略冲突就退出事务
    int ABORT = 3;
    // 策略冲突就使事务失败
    int FAIL = 4;
    // 忽略冲突
    int IGNORE = 5;
}
```

4.实现Database类

```
@Database(entities = {User.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

创建继承自 RoomDatabase 的抽象类，这个类时访问数据库的关键。
在这个抽象类中用注解配置与数据库中的表相关联的实体类，并提供返回 Dao 对象的抽象方法。

在 build.gradle 中配置数据库的Schema的导出路径，如下配置的导出路径为 app/schemas/

```
android {
    ...
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation": "$projectDir/schemas".toString()]
            }
        }
        ...
    }
    ...
}
```

如果不需要导出Schema，可以再第四步中使用如下配置：

```
@Database(entities = {User.class}, version = 1, exportSchema = false)
```

你应该存储导出的JSON文件，代表了你数据库schema的历史，在你的版本控制系统中，正如它允许创建老版本的数据库去测试。

为了测试这些migrations，添加 android.arch.persistence.room:testing 到你的测试依赖当中，并且把schema 位置当做一个asset文件添加，如下所示：

```
android {
    ...
    sourceSets {
        androidTest.assets.srcDirs += files("$projectDir/schemas".toString())
    }
}
```

测试包提供一个可以读取这些schema文件的MigrationTestHelper类。
它也是Junit4 TestRule类，所以它能管理创建的数据库。
如下代码展示了一个测试migration的例子：

```
@RunWith(AndroidJUnit4.class)
public class MigrationTest {
    private static final String TEST_DB = "migration-test";

    @Rule
    public MigrationTestHelper helper;

    public MigrationTest() {
        helper = new MigrationTestHelper(InstrumentationRegistry.getInstrumentation(),
                MigrationDb.class.getCanonicalName(),
                new FrameworkSQLiteOpenHelperFactory());
    }

    @Test
    public void migrate1To2() throws IOException {
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 1);

        // db has schema version 1. insert some data using SQL queries.
        // You cannot use DAO classes because they expect the latest schema.
        db.execSQL(...);

        // Prepare for the next version.
        db.close();

        // Re-open the database with version 2 and provide
        // MIGRATION_1_2 as the migration process.
        db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2);

        // MigrationTestHelper automatically verifies the schema changes,
        // but you need to validate that the data was migrated properly.
    }
}
```

当运行你app的测试时，你不应该创建一个完全的数据库如果你不测试数据库本身。Room允许你轻松的模仿数据访问层在测试当中。
这个过程是可能的因为你的DAOs不暴漏任何你数据库的细节。当测试你的应用，你应该创建模仿你的DAO类的假的实例。

测试你的数据库推荐的方法实现是写一个单元测试在Android设备上。因为这些测试不需要创建一个activity，他讲bicentennialUI单元测试快。
当装置你的测试用例时，你应该创建一个数据库的内存版本好让你的测试更密闭，如下所示：

```
@RunWith(AndroidJUnit4.class)
public class SimpleEntityReadWriteTest {
    private UserDao mUserDao;
    private TestDatabase mDb;

    @Before
    public void createDb() {
        Context context = InstrumentationRegistry.getTargetContext();
        mDb = Room.inMemoryDatabaseBuilder(context, TestDatabase.class).build();
        mUserDao = mDb.getUserDao();
    }

    @After
    public void closeDb() throws IOException {
        mDb.close();
    }

    @Test
    public void writeUserAndReadInList() throws Exception {
        User user = TestUtil.createUser(3);
        user.setName("george");
        mUserDao.insert(user);
        List<User> byName = mUserDao.findUsersByName("george");
        assertThat(byName.get(0), equalTo(user));
    }
}
```

定义转换类，@TypeConverter注解定义转换的方法。

```
public class DateConverter {

    @TypeConverter
    public static Date toDate(Long time) {
        return time == null ? null : new Date(time);
    }

    @TypeConverter
    public static Long toTime(Date date) {
        return date == null ? null : date.getTime();
    }
}
```

@TypeConverters注解，告知数据库要依赖哪些转换类。

```
@Database(entities = {User.class}, version = 1)
@TypeConverters({Converters.class})
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

使用这些转换器，您可以在其他查询中使用您的自定义类型，正如您将使用基本类型一样，如下代码所示：

```
@Entity
public class User {
    ...
    private Date birthday;
}
```

```
@Dao
public interface UserDao {
    ...
    @Query("SELECT * FROM user WHERE birthday BETWEEN :from AND :to")
    List<User> findUsersBornBetweenDates(Date from, Date to);
}
```

5.在Activity或Fragment类中为Database类声明和初始化对象

```
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();
```

> 注：在实例化AppDatabase对象的时候你应该遵循单例模式，因为每个Database实例都是相当昂贵的，而且你也很少需要多个实例。

生成数据库实例的其他操作

```
Room.databaseBuilder(context.getApplicationContext(),TaskDatabase.class, "name.db")
    .addCallback(new RoomDatabase.Callback() {
        // 第一次创建数据库时调用,但是在创建所有表之后调用的
        @Override
        public void onCreate(@NonNull SupportSQLiteDatabase db) {
            super.onCreate(db);
        }

        // 当数据库被打开时调用
        @Override
        public void onOpen(@NonNull SupportSQLiteDatabase db) {
            super.onOpen(db);
        }
    })
    .allowMainThreadQueries()// 允许在主线程查询数据
    .addMigrations()// 迁移数据库使用
    .fallbackToDestructiveMigration()// 迁移数据库如果发生错误,将会重新创建数据库,而不是发生崩溃
    .build();
```

## 数据库升级

1.新建要增加实体类。

2.在database中entities注解加入新增的实体类，同时版本号加一。

3.初始化数据库的地方增加addMigrations()方法，实现Migration类，例如：

```
AppDatabase db = Room.databaseBuilder(getApplicationContext(),AppDatabase.class, "database-name")
                .addMigrations(migration)
                .build();
```

```
private Migration migration = new Migration(1, 2) {

    @Override
    public void migrate(@NonNull SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Book` (`id` INTEGER,`name` TEXT, `owner` INTEGER, PRIMARY KEY(`id`))");
    }
};
```

表示从版本1升级到版本2，手动增加新增的表的sql。

## 嵌套对象

有时，您希望将一个实体或普通的以前的Java对象(POJO)作为数据库逻辑中的一个完整的整体来表示，即使该对象包含几个字段。
在这些情况下，您可以使用@Embedded来表示一个对象，您希望将其分解为表中的子字段。然后可以像对其他单个列一样查询嵌入式字段。

```
class Address {
    public String street;
    public String state;
    public String city;

    @ColumnInfo(name = "post_code")
    public int postCode;
}

@Entity
class User {
    @PrimaryKey
    public int id;

    public String firstName;

    @Embedded
    public Address address;
}
```

这样user表中的字段就包含了id, firstName, street, state, city, 和 post_code。

> 注意：嵌入式字段还可以包含其他嵌入式字段。

如果一个实体具有相同类型的多个内嵌字段，则可以通过设置前缀属性(prefix)使每个列保持惟一。然后将所提供的值添加到嵌入对象中每个列名的开头。

```
@Embedded(prefix = "foo_")
Coordinates coordinates;
```

## 和LiveData一起使用

添加依赖

```
// ReactiveStreams support for LiveData
implementation "android.arch.lifecycle:reactivestreams:1.0.0"
```

修改返回类型

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    public LiveData<List<User>> getAll();
}
```

## 和RxJava一起使用

添加依赖

```
// RxJava support for Room
implementation "android.arch.persistence.room:rxjava2:1.1.0"
```

修改返回类型

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    public Flowable<List<User>> getAll();
}
```

## 直接游标访问

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    public Cursor getAll();
}
```