<!-- toc -->

## 第七章 内容提供器

Content Provider 主要用于不同应用程序之间的数据共享功能，是Android 实现跨程序共享的数据的标准方式。

* 可以选择具体某一部分进行共享。

### :one: 运行时权限

用户无需再安装软件时授予所有权限，可以在使用过程中对某一项权限进行授权

* 普通权限无需手动操作，系统可以自动授予。

* 危险权限，需要手动点击授权。详见：[developer.android.com:权限组](https://developer.android.com/guide/topics/security/permissions?hl=zh-cn#perm-groups)

  > 每个危险权限都属于一个权限组。一旦用户授权了一个权限，**则该权限所属的权限组下的所有权限也会同时被授权。**

  如果所要的权限**不在**这张表中，则**需要**在AndroidManifest.xml中声明这些权限。
  
  
  
   :pushpin:总体步骤
  
  1. 获取内容的Uri
  
  2. 借助ContentResolver进行CRUD操作
  
     

以拨打电话为例。

```java
Button btn_call = findViewById(R.id.xxx);
btn_call.setOnClickListener(v -> {
   // 为了防止程序崩溃，代码放在异常捕获代码块中
    try{
        // 隐式Intent，内置的打电话的动作
        Intent intent = new Intent(Intent.ACTION_CALL);
        // 声明协议为tel
        intent.setData(Uri.parse("tel:10086"));
        startActivity(intent);
    }catch(SecurityException e){
        e.printStackTrace();
    }
});
```

接下来在AndroidManifest.xml声明权限

```xml
<user-permission android:name="android.permission.CALL_PHONE"/>
```

**这样的代码在6.0以上版本运行会失败（低于的可以）**

运行时权限的修改方式如下：

```java
Button btn_call = findViewById(R.id.xxx);
btn_call.setOnClickListener(v -> 
   // 首先检查用户是否已经已经授权了
   if(ContextCompat.checkSelfPermission(
       MainActivity.this, 
       Manifest.Permisson.CALL_PHONE) != 				            PackageManager.PERMISSON_GRANTED){
       // 未授权
       ActivityCompat.requestPermisson(this, 
       new String[]{Mainfest.Permisson.CALL_PHONE}, 1);
   }else{
       call();
   }
});

void call(){
     try{
        // 隐式Intent，内置的打电话的动作
        Intent intent = new Intent(Intent.ACTION_CALL);
        // 声明协议为tel
        intent.setData(Uri.parse("tel:10086"));
        startActivity(intent);
    }catch(SecurityException e){
        e.printStackTrace();
    }
}

@override
public void onRequestPermissonResult(int requestCode, String[] permissions, int[] gradtresults){
    switch(requestCode){
        case 1:
            if(gradtresults.length > 0 && gradtresults[0] == 
               PackageManager.PERMISSON_GRANTED)  call();
            else {
                // 失败提示
            }
    }
}
```

当前的授权情况

`ContextCompat.checkSelfPermission(Context context, int permisson);`

获取运行时权限

` ActivityCompat.requestPermisson(Activity activity, String[] Permissons, int requesCode);`

requestPermisson的回调函数`void onRequestPermissonResult(int requestCode, String[] permissions, int[] gradtResults)`

* requestCode: 请求时的代码，见`requestPermisson`

* grantResults: 请求结果。

### :two: 访问其他应用程序里的数据

ContentResolver：用于访问内容提供器的工具, 可以通过Content中的`getContentResolver()`方法来获取实例。

* CRUD是接受的是Uri对象

  > Uri对象有authority（一般是包名）和path两部分组成，另外还要加上协议声明
  >
  > `content://com.example.app/table`

* 有insert, delete, update, query等CRUD方法

查询操作：

```java
// 解析Uri对象
Uri uri = Uri.prase("content://com.example.app/table");
// 查询
// uri:表名, projection:列名， selection：where, selectionArgs:where参数，sortOrder：orderBy
Cursor cursor = getContent().getContentResover().query(
	uri, projection, selection, selectionArgs, sortOrder);
if(cursor != null){
    while(cursor.moveToNext()){
        String demo = cursor.getString(cursor.getComlumName("conlum1"));
    }
    cursor.close();
}
```

增加操作：

```java
ContentValues values = new ContentValues();
values.put("comlums1", 1);
values.put("comlums2", "two");
getContentResolver().insert(uri, values);
```

删除操作

```java
getContentResolver(url, "columns1 = ?", new String[] {"1"});
```

修改操作

```java
ContentValues values = new ContentValues();
values.put("comlums2", "new 2");
getContentResolver().update(uri, values, "conlumns2 = ? and conlumns3 = ?", new String[] {"two", "three"});
```

​                      

实例：查询联系人

权限

```xml
<uses-permisson android:name="android.permisson.READ_CONTACTS"</uses-permisson>
```

```java
// 前提已经申请 运行时 相关权限
private void readContracts(){
  Cursor cursor = new Cursor();
  try{
    cursor = getContentResovler().query(ContactsContract.CommonDatakinds.Phone.CONTENT_URI, null, null, null, null);
    if(cursor != null){
      while(cursor.moveToNext()){
        // 联系人姓名
        String displayName = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDatakinds.Phone.DISPLAY_NAME));
        // 联系人号码
        String displayNum  = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDatakinds.Phone.NUMBER));
      }
    }
  }
  catch(Exception e){
    e.printStackTrace();
  }
  finally{
     cursor.close();
  }
}
// 回调
```

### :three: 创建自己的内容提供器

为了实现跨程序共享数据，建议使用ContentProvider

自定义的方法：继承ContentProvider，需要将以下六个方法重写

```java
public class MyProvider extends ContentProvider {
    @Override
    public boolean onCreate() {
        // 在这里进行初始化，数据库创建、升级等
        // 返回true，创建成功；false，失败
        return false;
    }

    @Nullable
    @Override
    public Cursor query(Uri uri, String[] projection, String selection,  String[] selectionArgs,  String sortOrder) {
        // uri:表名, projection:列名， selection：where, selectionArgs:where参数，sortOrder：orderBy
        // 返回Cursor
        return null;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        return null;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }
}

```

> Uri的通配符：* 匹配任意长度的字符；# 匹配任意长的数字

利用UriMatcher类匹配Uri内容，以query方法为例

```java
public class MyProvider extends ContentProvider {
	public static final int TYPE_A = 1;
    public static final int TYPE_B = 2;
    public static final int TYPE_C = 3;
    public static UriMatcher uriMatcher;

    // 定义映射的编号
    static {
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        uriMatcher.addURI("com.example.app.provider", "table1", TYPE_A);
        uriMatcher.addURI("com.example.app.provider", "table2/#", TYPE_B);
        uriMatcher.addURI("com.example.app.provider", "table3", TYPE_C);
    }
    
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        switch (uriMatcher.match(uri)){
            case TYPE_A:
                // Operation
                break;
            case TYPE_B:
                // Operation
                break;
            default:break;
        }
        return null;
    }
}
```

getType()方法需要返回Uri对应的MIME类型

* 必须以`vnd`开头；

* 如果内容URI以路径结尾，则后接`android.cursor.dir/`, 如果以id结尾，则后接`android.cursor.item/`

* 最后接上`vnd.<authority>.<path>`

  如`content://com.package.one/table1`;

  :point_right: `vnd.android.cursor.dir/vnd.com.package.one/table1`

#### 注册

```xml
</application>
	<activity/>
	<provider
              
            android:name=".MyProvider"
            android:authorities="com.example.sqlite_demo.provider"
            android:enabled="true"
            android:exported="true"/>
</application>
```

name: 使用的类名

authorities：自定义

enabled：是否启用

exported：是否允许外部访问

