<!-- toc -->

## 第二章 Activity的基本用法

### findViewById()

`findViewById(id) `返回的是`View`对象 注意要向下转型

```
Button button = (Button) findViewById(R.id.button)
```

### Toast

Toast通过静态方法 maketext(context, content, druation)创建对象,通过show()方法展示

一般上下文传入本活动的实例即可 `AnACtivity.this`

内容传入字符串

时长有两个

* `Toast.LENGTH_SHORT` 短时
* `Toast.LENGTH_LONG ` 长时

### 菜单menu

* 在res目录下新建menu目录

* menu目录 右键 -> new -> Menu resource File

* 添加`<item>`标签

  ```xml
  <item
        android:id:"@+id/add_item"
        android:title:"xxx"/>
  ```

* 在活动中重写`onCreateoptionsMene(Menu menu)`方法

  ```java
  @Override
  public boolean onCreateOptionsMenu(Menu menu){
  	getMenuInflater().infalte(R.menu.main, menu);
      //第一个参数是布局文件,第二个是重写方法时传入的menu对象
  	return true;
  }
  ```

* 重写`onOptionsItemsSeleted(MenuItem item)`方法

  ```java
  @override
  public boolean onOptionsItemsSeleted(MenuItem item){
  	switch (item.getItemId()){
  		case R.id.xxx:
  			//TODO
  			break;
  		case R.id.yyy:
  			//TODO
  			break;
          defalut:
  	}
  	return true;
  }
  ```

完成后,在Activity标题栏的右边(如果未被隐藏)有三个点的图标

### 销毁活动

`finish()`  等同于`Back`键

### Intent

(意图)

* 用于各个组件之间的交互,它不仅可以指明**当前组件想要执行的动作**,也可以**传递数据**

* 一般用于启动活动,发送广播,启动服务等场景

* 分为**显式Intent和隐式Intent**

  (以下Intent用于启动)

#### 显式Intent

1. 建立Intent对象 

   ```java
   Intent intent = new Intent(Context packageContext ,Class<?> cls);
   ```

   第一个参数 用于启动活动的上下文, 第二个参数指定要启动的目标活动

2. `startActivity(intent)`

例如:

```java
Intent intent = new Intent(AnActivity.this , newActivity.class);
startActvity(intent);    
```

#### 隐式Intent

不明确指出想要启动的活动,而是通过指定一系列更加抽象的`action`和`category`信息,交由系统分析出"合适的活动"

* 在AndroidManifest.xml文件下的对应的`<activity>`标签下增加`<intent-filter>`的内容,可以指定合适的action和category,**只有同时满足两者才能实现intent**

  ```xml
  <activity Android:name=".xxx">
  	<intent-filter>
  		<action android:name="[包名].ACTION_START" />
          <category android:name="zzz"/>
      </intent-filter>
  </activity>
  ```

  * 需要如此声明Intent,直接将`action`的**字符串**传进去,表明想要启动`ACTION_START`这个活动

  ```java
  Intent intent = new Intent("[包名].ACTION_START");
  startActvity(intent);
  ```

  ​	默认的category是`android.intent.category.DEFAULT`

  * 通过`intent.addCategory(String category)`来增加category

#### 隐式Intent调用其他应用程序的活动

也可以调用其他应用程序的活动

 1. 调用系统浏览器打开网页

    ```java
    // in a listener
    Intent intent = new Intent(Intent.ACTION_VIEW);//Intent.ACTION_VIEW 自带
    intent,setData(Uri.prase("http://www.xxx.com"));
    startActivity(intent);
    ```

    也可以在<intetn-filter>内增加更多标签满足更精确的隐式Intent

    `<data android:scheme:"http">`只响应http协议

2. 调用系统拨打电话

```java
// in a listener
Intent intent = new Intent(Intent.ACTION_DIAL);//Intent.ACTION_VIEW 自带
intent.setData(Uri.prase("tel:10086"));
startActivity(intent);
```

`<data android:scheme:"tel">`只响应tel协议

#### 隐式Intent在活动之间传递信息

##### 1.传递数据给下一个活动

发送方:`intent.putExtra("key", value)`

```java
// 前略 创建intent实例
intent.putExtra("key", "A message");
```

接收方 `get<?>Extra(k)`

```java
// onCreate()函数内
Intent intent = getIntent(); // 获取用于启动Activity的Intent
String data = intent.getStringExtra("key");
```

注意get<?>Extra内填写相应的数据类型

##### 2.返回数据给上一个活动

Activity内另外一个启动活动的方法`startActivityForResult(Intent intent)`,期望新的活动销毁时能够返回一个结果给上一个活动.

原活动:

​	1. startActivityForResult(Intent intent , int requestCode ), **需要声明请求码辨别来源**

```Java
// 创建好了一个Intent实例intent
startActivityForResult(intent, 1);
```

​	2.重写`onActivityResult()`用于接收

```java 
@override
protected void onActivityResult(int requestCode , int resultCode, Intent data){
	switch(requestCode){ // 判断数据来源
		case 1:
			if (resultCode == RESULT_OK){
				//TODO
				
			break;
        default:														}		
	}
}
```



新活动:

Intent构造方法参数为空

```java
//在某事件监听器内或onBackPress()内(监听返回键)
...
Intent intent = new Intent();
// 可以使用putExtra()添加传递的信息
intent.setResult(RESULT_OK, intent);
// 销毁 onBackPress()不需要
finish();        
```

setResult的第一个参数只能有两种选择`RESULT_OK`, `RESULT_CANCEL`

### 活动生命周期

#### 活动状态

* 运行状态: 返回栈
* 暂停状态 :不是活动态但是仍可见(对话框遮档)
* 停止状态: 不在栈顶位置,也完全不可见
* 销毁状态: 从返回栈中移除

#### 生存期

* onCreate(): 活动第一次被创建 应该完成活动的初始化操作 如加载布局,绑定事件等
* onStart() 不可见变可见时
* onResume() 准备好与用户交互 此时活动必然处于栈顶
* onPause() 在系统准备启动新活动是调用 用于保存关键数据 但一定要快
* onStop()  活动完全不可见前调用 
* onDestroy() 销毁之前调用 之后为销毁态
* onRestart() 停止状态 --> 运行态

#### 生存期

* 完整生存期 																

  onCreate()  -->  onDestroy()

* 可见生存期 对于用户可见时,即使无法与用户交互 

  onStart()  --> onStop()

* 前台生存期  可见而且可以与用户交互			        

  onResume()  --> onPause()

#### 活动被回收了怎么办 onSaveInstanceState()

一个活动到另外一个新的活动,原活动处于停止状态,可能被系统回收.从新活动返回时仍可以重新打开原活动,

执行`onCreate()`方法.但原来的数据被丢失.

`onSaveInastanceState(Bundle outside)`可以临时保存状态.

```java
@override
protected void onSaveInstanceState(Bundle outside){
	super.onSaveInstanceState(outside);
	outside.putString("A_key", "value");
}
```

在`onCreate()`中取出:

```java
onCraete(Bundle savedinstanceState){
	super.onCreate(savedinstanceState);
	setContentView(xxx);
	
	if(savedinstanceState !== null){
		String tempDate = savedinstanceState.getString("A_key");
	}
}
```

### 活动的启动模式

```xml
<activity android:name=".MainActivity"
    android:launchMode=xxx
</activity>
```

1. standard(默认): 新的活动都在栈顶, 不管这个活动在返回栈在是否存在
2. singeTop: 新活动启动时发现栈顶的活动已经是该活动, 直接使用它, 不创建新活动
3. singleTask: 使得某个活动在栈中只存在一个实例
4. singleInstance: 单独创建一个返回栈管理该活动, 不管是那个应用程序访问这个活动, 都共用这个返回栈, 从而实现共享