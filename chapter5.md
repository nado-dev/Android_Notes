<!-- toc -->

## 第五章 广播和广播接收器BroadCast Receiver

### 分类

* 标准广播  完全异步，所有广播接收器都同一时刻接收到消息
* 有序广播  按序传播，可以被截断

### 接收系统广播

* 广播接收器可以选择自己感兴趣的广播进行注册

  注册广播的方式有**动态注册**(代码中注册)， **静态注册**（AndroidManifaest.xml）两种

* 创建广播接收器的方法: **新建一个类, 继承自`BroadcastReceiver`类即可,** 广播到来时`onReceive()`方法可以得到触发



#### 动态注册

动态注册只有当程序启动后才会执行

MainActivity.java

```java
public class MainActivity extends AppCompatActivity{
    //
    private IntentFilter intentFilter;
    //
    private NetworkChangeReceiver networkChangeReciver;
    
    @override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.xxx);
        
        // 建立IntentFilter对象
        intentFilter = new IntentFilter();
        // 想要监听什么广播,在addAction方法在添加
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver = new NetWorkChangeReceiver();
        // 注册广播接收器
        resigerReciver(networkChangeReceiver, intentFilter);
    }
    
    protected void onDestory(){
        super.onDestory();
        // 取消已经注册的监听器
        unregsiterReceiver(networkChangeReceiver)
    }
    // 新建一个接收器,注意onReceive的参数列表
    class NetWorkChangeReciver extends BroadcastReceiver{
        @override
        public void onReceive(Context context, Intent intent){
            Toast.makeText(...).show();
        }
    } 
}
```

注册监听器

` resigerReciver(sub_BroadcastReceiver receiver, intentFilter intentFilter);`

注销监听器

`unresigerReciver(sub_BroadcastReceiver receiver)`; 

扩展:获取当前网络状态的方法

```java
// 获取系统服务 网络连接管理器
ConnectivityManager connectivityManager = getSystemManager(Context.CONNECTIVITY_SERVER);
// 实例化网络状态类
NetworkInfo netwotkInfo = connectivityManager.getActivenetworkInfo();
//--> log.v(TAG, networkInfo.isAvailable());
```

注册权限:

AndroidManifest.xml

```xml
<uses-permission android:name:"android.permission.ACCESS_NETWORK_STAT"/>
```

#### 静态注册

静态注册可以程序在未启动的情况下接收到广播

1. `new->other->Broadcast Receiver`

2. 命名, 选择`Enable`(启用), `Exported` (允许接收程序外的广播), 打开后在`onReceive`内重写逻辑

   AndroidManifest.xml的 `<application>`标签内多了一个**`<recevier>`标签**, 功能类似于`<activity>`

3. 在`<recevier>`内声明要监听的广播

   ```xml
   <receiver
        android:name:".已声明的广播接收器类"
             ...>
       <intent-inflater>
       	<action android:name="xxx">
       </intent-inflater>
   </receiver>
   ```

```
### 发送自定义广播

#### 发送标准广播

设已经有一个`MyBroadcastReceiver`类, 继承了`BroadcastReceiver`类, 并且实现了简单的`onReceive`方法

AndroidManifest.xml

​```xml
<receiver
     android:name:".MyBroadcastReceiver"
          ...>
    <intent-inflater>
    	<action android:name="com.example.broadcast.MY_BROADCAST">
    </intent-inflater>
</receiver>
```

意思是让`MyBroadcastReceiver`类可以接收一条`com.example.broadcast.MY_BROADCAST`广播

现在在一个活动内, 假设一个按钮触发了发送广播的事件

```java
public void onClick(View v){
    // 创建intent实例, 构造方法传入广播名
    Intent intent = new Intent("com.example.broadcast.MY_BROADCAST");
    // 发送广播
    sendbroadcast(intent);
}
```

#### 发送有序广播

有先后接受顺序的广播, 可以通过**对广播接收器设置优先级**实现

发送有序广播只要将`sendBroadcast(intent)`改为`sendOrderedBroadcast(intent, null)`

第二个传入权限的字符串, 一般null即可

对接收器优先级`android:priority="xxx"`

```xml
<receiver
     android:name:".MyBroadcastReceiver"
          ...>
    <intent-inflater
         android:priority="100"
                     >
    	<action android:name="com.example.broadcast.MY_BROADCAST">
    </intent-inflater>
</receiver>
```

如果高优先级的接收器捕捉到了广播, 可以在处理逻辑内调用`abortBroadcast()`函数, 低优先级的无法收到这条广播.

* 以上写法发送的均为全局广播

#### 发送本地广播

```java
public class MainActivity extends AppCompatActivity{
    private IntentFilter intentFilter; 
    private LocalReciver localReciver;
    // 新增的本地广播管理器
    private LocalBroadcastManager localBroadcastManager;
    
    @override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.xxx);

        // 不同之处 获取实例
        localBroadcastManager = LocalBroadcastManager.getInstance(this);
        intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.broadcast.MY_BROADCAST");
        localReciver = new LocalReciver();
        // 不同之处 建立本地广播接收器
        localBroadcastManager.resigerReciver(networkChangeReceiver, intentFilter);
        
        Button btn = (Button)findViewById(R.id.xxx);
        btn.setOnClickListener(View.OnClickListener(){
            public void onClick(){
                 Intent intent = new Intent("com.example.broadcast.MY_BROADCAST");
                // 不同之处 发送本地广播
                localBroadcastManager.sendBroadcast(intent);
            }
        });
    }
    
    protected void onDestory(){
        super.onDestory();
        // 不同之处 销毁本地广播接收器
        localBroadcastManager.unregsiterReceiver(networkChangeReceiver)
    }
    class LocalReciver extends BroadcastReceiver{
        @override
        public void onReceive(Context context, Intent intent){
            Toast.makeText(...).show();
        }
    } 
}
```

### 最佳实践  强制下线

增加一层`BaseActivity`, 里面实现广播接收器功能和活动队列管理功能