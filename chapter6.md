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

   接收一个context参数 包名当作文件名

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

