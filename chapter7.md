<!-- toc -->

## 第七章 内容提供器

Content Provider 主要用于不同应用程序之间的数据共享功能，是Android 实现跨程序共享的数据的标准方式。

* 可以选择具体某一部分进行共享。

### 运行时权限

用户无需再安装软件时授予所有权限，可以在使用过程中对某一项权限进行授权

* 普通权限无需手动操作，系统可以自动授予。

* 危险权限，需要手动点击授权。详见：[developer.android.com:权限组](https://developer.android.com/guide/topics/security/permissions?hl=zh-cn#perm-groups)

  > 每个危险权限都属于一个权限组。一旦用户授权了一个权限，**则该权限所属的权限组下的所有权限也会同时被授权。**

  如果所要的权限**不在**这张表中，则**需要**在AndroidManifest.xml中声明这些权限。

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
       Manifest.Permisson.CALL_PHONE) != 				            PackageManager.PERMISSON.GRANTED){
       // 未授权
       ActivityCompat.requestPermisson(this, 
       new String[]{Mainfest.permisson.CALL_PHONE}, 1);
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
               PackageManager.PERMISSON.GRANTED)  call();
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

* gradtResults: 请求结果。

### 访问其他应用程序里的数据

ContentResolver：用于访问内容提供器的工具, 可以通过Content中的Content中的`getContentResolver()`方法来获取实例。

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