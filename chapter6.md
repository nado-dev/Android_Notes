<!-- toc -->

## 第六章 持久化存储

文件存储, SharedPreference, 数据库

### 文件存储

不推荐, 适用于简单无格式文本

#### 写

类似Java提供的文件存储方法

context 类中提供了一个`openFileOutput()`方法, 接收两个参数, 返回FileOutputStream对象     

* 文件名 String
* 文件操作模式 `MODE_PRIVATE` (覆盖) `MODE_APPEND`(追加)

注意, 不定义路径, 所有文件都保存在`/data/data/<packagename>/files/`目录下

```java
public void save(){
    String data = "data";
    FileOutputStream out = null;
    BufferWriter writer= null;
    try{
        out = openFileOutput("data", Context.MODE_PRIVATE);
        writer = new BufferWriter(new OutputStreamWriter(out));
        writer.write(data);
    }catch(IOException e){
        e.printStacktrace();
    }finally{
        try{
            if(writer != null){
                writer.close();
            }catch(IOException e){
                e.printStacktrace();
            }
        }
    }
}
```

#### 读

Context类中还有`openFileInput()`方法, 只接受一个参数文件名String, 返回FileInputStream对象

```java
public String load(){
    StringBulider content = new StringBulider();
    FileInputStream in = null;
    BufferReader reader = null;
    try{
        in = openFileInput("data");
        reader = new BufferReader(new InputStreamReader(in));
        String singleLine = "";
        while((singleline = reader.readline())!= null){
            content.append(singleline);
        }
    }catch(IOException e){
        e.printStacktrace();
    }finally{
        try{
            if(reader != null){
                reader.close();
            }catch(IOException e){
                e.printStacktrace();
            }
        }
    }
    return content.toString();
}
```

### SharedPreference存储

* 键值对存储
* 支持指定数据类型
* xml文件保存

#### 获取SharedPreference对象的方法

1. Context类中的`getSharedPreferences()`方法

   * 第一个参数, 文件名String 不存在则会创建
   * 第二个参数 打开模式 只有`MODE_PRIVATE`可选

2. Activity类中的`getSharedPreferences()`方法

   ​	只有一个参数 打开模式 只有`MODE_PRIVATE`可选

   ​	会将自己的活动类名当作文件名

3. PreferenceManager类中的`getDefaultSharedPreferences()`方法

   接收一个context参数 **包名当作文件名**

#### 写

```java
onClick(){
    // 1. 调用SharePreferences对象中的edit()方法获取一个SharePreferences.Editor对象
    SharePreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
    
    // 2. 向SharePreferences.Editor对象中添加数据
    editor.putString("key_String", "value_String");
    editor.putBoolean("key_boolean", true);//ect...    	
    
    // 3. apply()方法提交
    editor.apply();
}
    
```

#### 读

```java
onClick(){
    // 1.获取SharePreferences对象
    SharedPreferences pref = getSharedPreferences("data", MODE_PRIVATE);
    
    // 2.根据不同的数据用不同的get方法(类似Intent取值)
    	// 第一个参数key 第二个参数默认值,当key找不到对应时的默认值
    String string = pref.getString("key_String", "default String");
    boolean bool = pref.getBoolean("key_boolean", false);
}
```

* `clear()`方法，清除文件内容

### SQLite数据库存储技术

抽象类`SQLiteOpenHelper`

* 需要自行实现一个帮助类, 重写`onCreate()`和`onUpgrade()`实现创建 更新数据库的逻辑

* 两个重要的实例方法`getReadableDatabase()`和`getWritableDatabase()`

  可以创建或者打开数据库, 返回一个可以对数据库进行操作的对象

* 构造方法接收四个参数 context,  数据库名, 自定义的Cursor(null), 数据库版本号

#### onCreate()建表

MainActivity

```java
  btn_createTable = findViewById(R.id.create_table);
        MyDataBaseHelper dbHelper = new MyDataBaseHelper(this, "BookStore.db", null, 1);
        btn_createTable.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
        });
```



```java
public class MyDataBaseHelper extends SQLiteOpenHelper {

    // 建表SQL语句
    public static final String CREATE_BOOK = "create table Book("
            + "id integer primary key autoincrement,"
            + "author text,"
            + "price real,"
            + "pages integer,"
            + "name text)";

    // context副本便于其他地方利用
    private Context mContext;

    public MyDataBaseHelper(@Nullable Context context
            , @Nullable String name
            , @Nullable SQLiteDatabase.CursorFactory factory
            , int version)
    {
        super(context, name, factory, version);
        mContext = context;
    }

    // 创建逻辑有关操作
    @Override
    public void onCreate(SQLiteDatabase db) {
        // execSQL()执行SQL语句
        db.execSQL(CREATE_BOOK);
        Toast.makeText(mContext, "Created!", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}

```

#### adb工具

1. 将`AndroidSDK\platform-tools`添加进入环境变量
2. cmd `adb shell`进入到设备的控制台

连接到mumu `adb connect 127.0.0.1:7555`

注意:mumu模拟器并无sqlite3文件, 导入方法见[Android 模拟器 sqlite3命令 not found 解决办法](https://blog.csdn.net/sjtuzhou/article/details/89365178)

##### sqlite3 命令总结

```shell
sqlite3 [DatabaseName] #进入数据库
.table #查看表
.schema #查看数据库建立语句
```

#### onUpgrade()升级表

一般在增加新表时用到

`onUpgrade()`在`SQLOpenHelper`()传入的构造函数的第四个**参数版本号比之前的大**时执行

```java
 public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("drop table if exists Book");
        db.execSQL("drop table if exists Category");
        onCreate(db);
    }
```

#### 添加数据

`getWritableDatabase()`和`getReadableDatabase()`返回SQLiteDatabase对象, 该对象可以方便地进行CRUD操作

添加数据使用`insert`方法

* 参数1: 表名String
* 参数2: 指定在未添加数据的情况下为某些可为空的咧自动赋值为空 一般传入NULL即可
* 参数3: 一个`ContentValues`对象, 通过`put`方法向其中填入值, 完成一个元组 (记录)

```java
 public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();
                values.put("name", "nmsl");
                values.put("author", "sunxiaochuang");
                values.put("pages", 12);
                values.put("price", 6324);

                long result =db.insert("Book",null,values);
                if (result > 0){
                   showSucceed();//showToast
                }
            }
```

#### 更新数据

`update()`方法

* 参数1: 表名String

* 参数2: 一个`ContentValues`对象, 通过`put`方法向其中填入值, 更新的部分

* 参数3: `where`语句  若条件填入`?`, 意为做占位符, 具体条件见参数4

  `name=?`

* 参数4: 提供一个字符串数组,指定每个占位符的内容

  `new String[]={"BookName"}`

```java
btn_update.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                ContentValues values = new ContentValues();
                values.put("price", 666);

                int res = db.update("Book", values, "name=?", new String[]{"nmsl"});
                if (res > 0){
                    showSucceed(true);
                }
            }
        });
```

#### 删除数据

`delete()`方法

* 参数1: 表名String
* 参数2: `where`语句  若条件填入`?`, 意为做占位符, 具体条件见参数3
* 参数3: 提供一个字符串数组,指定每个占位符的内容

```java
 btn_delete.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                db.delete("Book", "id>?", new String[]{"3"});
            }
        });
```

#### 查询数据

`query()`方法

query(table, columns, selection, selectionArgs, groupBy, having, orderBy, limit)方法各参数的含义：

* table：表名。相当于select语句from关键字后面的部分。如果是多表联合查询，可以用逗号将两个表名分开。
* columns：要查询出来的列名。相当于select语句select关键字后面的部分。
* selection：查询条件子句，相当于select语句where关键字后面的部分，在条件子句允许使用占位符“?”
* selectionArgs：对应于selection语句中占位符的值，值在数组中的位置与占位符在语句中的位置必须一致，否则就会有异常。
* groupBy：相当于select语句group by关键字后面的部分
* having：相当于select语句having关键字后面的部分
* orderBy：相当于select语句order by关键字后面的部分，如：personid desc, age asc;
* limit：指定偏移量和获取的记录数，相当于select语句limit关键字后面的部分。

返回一个Cursor对象, 所有数据从此取出

 `getString(int Colum)`  return Cursor

`getColumIndex(String ColumBName)`  return int(Colum Index)

```java
 btn_update.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db = dbHelper.getWritableDatabase();
                Cursor cursor = db.query("Book", null, null, null, null, null, null);
                if (cursor.moveToFirst()){
                    do{
                        String name = cursor.getString(cursor.getColumnIndex("name"));
                        Log.d("MainActivity", name);
                    }while (cursor.moveToNext());
                }
                cursor.close();
            }
        });
```

注意要关闭Cursor对象 

cursor.close();

****

注意:以上所有操作均可以属于SQL语句执行

增删改: `db.execSQL(String SQLsequence, String[] whereArgs)`

查找: `db.rawQuery(String SQLsequence)`

### LitePal操作数据库

LitePal一种是使用关系映射模式管理数据库的工具

#### 配置LitePal

1. 添加依赖

	```
	implementation 'org.litepal.android:core:1.4.1'
	```

2. `app/src/main` -> New -> Diretory     创建`assets`目录

3. `assets`  -> new -> File    创建``litepal.xml`

   ```xml
   <?xml version="1.0" encoding="utf-8" ?>
   <litepal>
       <dbname value="BookStore"></dbname>
       <version value="2"></version>
       
       <list></list>
   </litepal>
   ```

   其中` dbname`指数据库名

   ​	`version`指版本，可以用于修改表结构后实现版本控制

   `	list`指所有映射模型（待添加）

4. AndroidManifest.xml

   ```xml
   <application
                android:name="org.litepal.LitePalApplication"/>
   ```

#### 创建表

1. 新建实体类Book_new 定义各种属性(列), getter和setter

```java
public class Book_New extends DataSupport{
    private int id;
    private String author;
    private double price;
    private int pages;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

   //ect
}

```

2. 添加映射关系

   ```xml
   <list>
           <mapping class="com.example.sqlite_demo.Database.Book_New"></mapping>
   </list>
   ```

3. 调用`getDatabase()`

   ```java
    btn_createDB.setOnClickListener(new View.OnClickListener() {
               @Override
               public void onClick(View v) {
                   LitePal.getDatabase();
               }
           });
   ```

   

#### 更新表

比如修改表的结构(列)

1. 直接在相关的实体类中修改属性
2. 版本号+1

#### 添加数据

1. 建立实体类对象

2. 使用set方法添加属性

3. 调用`save`方法保存

   ```java
    btn_addDB.setOnClickListener((v) -> {
               Book_New book= new Book_New();
               book.setAuthor("lilaoba");
               book.setName("chibaba");
               book.setPages(6324);
               book.setPrice(6325);
   
               book.save();
           });
   ```

#### 更新数据

1. 建立实体类对象

2. 使用set方法添加需要更改的属性

3. 使用`updateAll()`方法, 传入相应的where子句进行更新

   * 若不传入任何参数, 对所有数据有效

   * 第一个参数传入where子句, 具体条件用 ? 占位

   * 后面的参数依次补充 ? 的内容

     

   ```java
   btn_updateDB.setOnClickListener((v)->{
               Book_New book = new Book_New();
               book.setPrice(1111.12);
               book.setPages(8888);
   
               book.updateAll("author=? and id=?", "lilaoba", "1");
           });
   ```

   ****

   若要将某个属性恢复为默认值  比如String的null,  int的0等, 不可以直接使用上面的方法传这些参数

   应该使用专门的`setToDefault()`方法,  传入列名String

   ```java
   book.setToDefault("author");
   book.updateAll("pages=?", "12");
   // 将pages=12的所有记录的author更新为默认值
   ```

   ****

   另外 调用过save方法的语句和查询出来的对象 可以直接使用对此对象修改后再调用save方法

   #### 删除数据

   类似于更新

   调用DataSupport里的静态方法`deleteAll`

   * 参数1: 用于指定删除指定表的数据
   * 参数2: where子句和占位符
   * 其他: 占位符补充

   ```java
   btn_updateDB.setOnClickListener((v)->{
               DataSupport.deleteAll(Book_new.class, "author=? and id=?", "lilaoba", "1");
           });
   ```

   

#### 查询数据

基本操作

```java
List<Book_New> books = DataSupport.findAll(Book.class);
```

返回所有记录的实例化列表  相当于 `select * from Book_New`

****

特殊位置

第一条记录

```java
Book firstbook = DataSupport.findFirst(Book.class);
```

最后一条记录

```java
Book lastbook = DataSupport.findLast(Book.class);
```

****

连缀查询

```java

select(String colums);// 选列

where(String sql, sqlArrs);// where子句

order(String seq);// 排序 相当于order by 关键字

limit(int num);// 指定查询结果的数量

offset(int offset);// 偏移量

find(xx.class);//相当于 from [xx表]
```

例如

```java
List<Book> books = DatabaseSupport.select("name", "id")
    .where("pages>?", "10")
    .order("id decs")
    .limit(10)
    .offset(1)
    .find(Book.class);
```

****

SQL原生查询

```java
Cursor cursor = DatabaseSupport.findBySQL("select * from Book where name=?", "lilaoba");
```

要经过迭代才能使用,参见``SQLiteDatabase`[查看](#查询数据)