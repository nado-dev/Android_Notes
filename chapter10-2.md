<!-- toc -->

## 第十章-2 服务

服务(Service)是Android中实现**后台运行**的方案，适合去执行不需要与用户交互而且要求长期运行的任务。服务的运行**不依赖任何用户界面**，即使任务被切换到后台，或者用户打开了另一个应用程序，服务仍然能够保持正常运行。

​	服务**并不是运行在单独的进程中**，而是依赖创建服务时的进程。当某个应用程序被杀掉时，所有依赖于该进程的服务都会被杀掉。

​	服务所有的代码都运行在主线程中，通常需要在服务的内部手动创建子线程并在此执行相应的任务，否则可能会出现主线程被阻塞住堵塞现象。

### 创建基本服务

new -> Service -> Service。Enable：是否启用；Exported：是否允许其他程序访问。 

<img src=".\chapter10-2.assets\image-20200704151334052.png" alt="image-20200704151334052" style="zoom: 67%;" />

创建后的Manifest.xml

```xml
<application
        android:name=".MyApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:requestLegacyExternalStorage="true"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <service
            android:name=".MyService"
            android:enabled="true"
            android:exported="true"></service>
    ...
</application>    
```

MyService.java

```java
public class MyService extends Service {
    public MyService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}
```

* onCreate() 服务创建时调用
* onStartCommand() 每次服务启动时：一旦启动就执行的动作
* onDestroy()服务销毁时调用：回收资源

Context中启动服务：

```java
Intent intent = new Intent(this, MyService.class);
startService(intent);
```

Context中关闭服务：

```java
Intent intent = new Intent(this, MyService.class);
stopService(intent);
```

Service也提供了方法

```
stopSelf();
```

### 活动和服务进行通信

此前介绍的方法活动和服务并不能彼此控制，联系不太紧密。

如果希望活动指挥服务，需要借助`onBind()`方法

-决定何时下载，查看下载进度

1. **继承Bind类**，在内部定义相关的控制方法；
2. 实例作为成员变量，
3. 作为`onBind()`的返回值

MyService.java

```java
public class MyService extends Service {
    private DownloadBinder downloadBinder = new DownloadBinder();

    public MyService() {}

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return downloadBinder;
    }

    class DownloadBinder extends Binder {
        public void starDownload() {
            Log.d("MyService", "starDownload");
        }

        public int getProgress() {
            Log.d("MyService", "getProgress");
            return 0;
        }
    }
	...
}
```

ServiceActivity.java

1. 首先创建ServiceConnection匿名类，里面重写`onServiceConnected()`方法和`onServiceDisconnected()`的方法，这两个方法分别在活动与服务连接时和连接断开时调用。	

2. 再将`onServiceConnected()`的参数中的`IBinder service`类向下转型为MyService.DownloadBinder。然后可以调用这个实例内部定义的一些方法实现控制。

3. 正式绑定

   ```java
    Intent intent = new Intent(this, MyService.class);
    bindService(intent, connection, BIND_AUTO_CREATE);
   ```

   第二个参数为ServiceConnection的实例，第三个参数`BIND_AUTO_CREATE`表示活动和服务绑定后自动创建服务，**会执行onCreate()方法，但onStartCommand()方法不会执行**。

4. 解除绑定

   ```java
   unbindService(connection);
   ```

   ServiceActivity.java:

   ```java
   public class ServiceActivity extends AppCompatActivity {
      ...
       private Button btn_bindService;
       private Button btn_unbindService;
       private TextView txt_tips2;
       private MyService.DownloadBinder downloadBinder;
       private ServiceConnection connection = new ServiceConnection() {
           @Override
           public void onServiceConnected(ComponentName name, IBinder service) {
               downloadBinder = (MyService.DownloadBinder)service;
               downloadBinder.starDownload();
               downloadBinder.getProgress();
           }
   
           @Override
           public void onServiceDisconnected(ComponentName name) {
   
           }
       };
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           ...
           btn_bindService = findViewById(R.id.bind_service);
           btn_unbindService = findViewById(R.id.unbind_service);
           txt_tips = findViewById(R.id.tips_service);
           txt_tips2 = findViewById(R.id.tips_service2);
   
         	...
           btn_bindService.setOnClickListener(v->mBindService());
           btn_unbindService.setOnClickListener(v->mUnBindService());
       }
   
       private void mUnBindService() {
           unbindService(connection);
       }
   
       private void mBindService() {
           Intent intent = new Intent(this, MyService.class);
           bindService(intent, connection, BIND_AUTO_CREATE);
       }
   	...
   }
   
   ```

   ### 服务的生命周期

   > 转载自https://www.cnblogs.com/huihuizhang/p/7623760.html

   ![Life Circle of Service](https://images2017.cnblogs.com/blog/896629/201710/896629-20171003150906568-1811048252.png)

1). 被启动的服务的生命周期：如果一个Service被某个Activity 调用 Context.startService 方法<u>启动</u>，那么不管是否有Activity使用bindService绑定或unbindService解除绑定到该Service，该Service都在后台运行。**如果一个Service被startService 方法多次启动，那么onCreate方法只会调用一次，onStart将会被调用多次**（对应调用startService的次数），并且系统只会创建Service的一个实例（因此你应该知道只需要一次stopService调用）。该Service将会一直在后台运行，**而不管对应程序的Activity是否在运行**，直到被调用stopService，或自身的stopSelf方法。当然如果系统资源不足，android系统也可能结束服务。

2). 被绑定的服务的生命周期：如果一个Service被某个Activity 调用 Context.bindService 方法<u>绑定启动</u>，不管调用 bindService 调用几次，onCreate方法都只会调用一次，同时onStart方法始终不会被调用。当连接建立之后，Service将会一直运行，除非调用Context.unbindService 断开连接或者之前调用bindService 的 Context 不存在了（如Activity被finish的时候），系统将会自动停止Service，对应onDestroy将被调用。

3). :star: **被启动又被绑定的服务的生命周期**：如果一个Service又被启动又被绑定，则该Service将会一直在后台运行。并且不管如何调用，onCreate始终只会调用一次，对应startService调用多少次，Service的onStart便会调用多少次。调用unbindService将不会停止Service，而必须调用 stopService 或 Service的 stopSelf 来停止服务。

4). 当服务被停止时清除服务：当一个Service被终止（1、调用stopService；2、调用stopSelf；3、不再有绑定的连接（没有被启动））时，onDestroy方法将会被调用，在这里你应当做一些清除工作，如停止在Service中创建并运行的线程。

特别注意：

1、你应当知道在调用 bindService 绑定到Service的时候，你就应当保证在某处调用 unbindService 解除绑定（尽管 Activity 被 finish 的时候绑定会自动解除，并且Service会自动停止）；

2、你应当注意 使用 startService 启动服务之后，一定要使用 stopService停止服务，不管你是否使用bindService；

3、:star: **同时使用 startService 与 bindService 要注意到，Service 的终止，需要unbindService与stopService同时调用，才能终止 Service，不管 startService 与 bindService 的调用顺序**，如果先调用 unbindService 此时服务不会自动终止，再调用 stopService 之后服务才会停止，如果先调用 stopService 此时服务也不会终止，而再调用 unbindService 或者 之前调用 bindService 的 Context 不存在了（如Activity 被 finish 的时候）之后服务才会自动停止；

4、当在旋转手机屏幕的时候，当手机屏幕在“横”“竖”变换时，此时如果你的 Activity 如果会自动旋转的话，旋转其实是 Activity 的重新创建，因此旋转之前的使用 bindService 建立的连接便会断开（Context 不存在了），对应服务的生命周期与上述相同。

5、在 sdk 2.0 及其以后的版本中，对应的 onStart 已经被否决变为了 onStartCommand，不过之前的 onStart 任然有效。这意味着，如果你开发的应用程序用的 sdk 为 2.0 及其以后的版本，那么你应当使用 onStartCommand 而不是 onStart。

**生命周期方法说明**

onStartCommand()
当另一个组件（如 Activity）通过调用 startService() 请求启动服务时，系统将调用此方法。一旦执行此方法，服务即会启动并可在后台无限期运行。 如果您实现此方法，则在服务工作完成后，需要由您通过调用 stopSelf() 或 stopService() 来停止服务。（如果您只想提供绑定，则无需实现此方法。）

onBind()
当另一个组件想通过调用 bindService() 与服务绑定（例如执行 RPC）时，系统将调用此方法。在此方法的实现中，您必须通过返回 IBinder 提供一个接口，供客户端用来与服务进行通信。请务必实现此方法，但如果您并不希望允许绑定，则应返回 null。

onCreate()
首次创建服务时，系统将调用此方法来执行一次性设置程序（在调用 onStartCommand() 或 onBind() 之前）。如果服务已在运行，则不会调用此方法。

onDestroy()
当服务不再使用且将被销毁时，系统将调用此方法。服务应该实现此方法来清理所有资源，如线程、注册的侦听器、接收器等。 这是服务接收的最后一个调用。

### 前台服务

后台服务的优先级较低，系统资源不足时可能会被回收。如果需要保持服务一直保持运行，可以考虑使用前台服务。形式是通知栏有一个类似通知的提示。（也有可能用于其他用途）

只需在**服务里的onCreate方法**里添加如下语句：

```java
 public void onCreate() {
        super.onCreate();
        Log.d("MyService", "onCreate");
        Intent intent = new Intent(this, ServiceActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
        Notification notification = new NotificationCompat.Builder(this,"default")
                .setContentTitle("通知")
                .setContentText("正文正文正文正文正文正文")
                .setWhen(System.currentTimeMillis())
                .setSmallIcon(R.mipmap.ic_launcher_round)
          .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                .setContentIntent(pi)
                .build();
        startForeground(1, notification);
    }
```

前面与创建通知的方法完全相同，最后需要调用`startForeground(int id, Notification notification)`创建前台服务

### IntentService

服务里需要处理耗时操作，如果在主线程则有可能出现ANR错误，故要开启子线程处理（在服务中一般在onStartCommand内开启子线程）操作。**但这种服务一旦启动后就会一直处于运行状态**，必须调用stopService()或者stopSelf()进行操作使其停止。

```java
public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d("MyService", "onStartCommand");
    	new Thread(()->{
           // 具体处理
            stopSelf();
        }).start();
        return super.onStartCommand(intent, flags, startId);
    }
```

为了避免忘记调用start()开启线程者关闭服务，专门为此提供了一个IntentService类，解决前面的问题。

**集开启线程和自动停止于一身。**

MyIntentService.java

```java
public class MyIntentService extends IntentService {

    /**
     * Creates an IntentService.  Invoked by your subclass's constructor.
     */
    public MyIntentService() {
        super("MyIntentService");
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        Log.d("MyIntentService", String.valueOf(Thread.currentThread().getId()));
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d("MyIntentService", "destroyed");
    }
}
```

注册Service：

```xml
<service android:name=".Service.MyIntentService"/>
```

![image-20200705160809880](.\chapter10-2.assets\image-20200705160809880.png)



### 实例 下载

1. 定义回调接口

   DownloadListener.java

   ```java
   package com.example.sqlite_demo.Interface;
   
   /**
    * 下载回调接口
    * 用于监听各种状态的监听和回调
    */
   public interface DownloadListener {
   
       /**
        * 进度百分比回调方法
        * @param progress 进度(0-100)
        */
       void onProgress(int progress);
   
       /**
        * 成功回调方法
        */
       void onSuccess();
   
       /**
        * 失败回调接口
        */
       void onFailed();
   
       /**
        * 暂停回调方法
        */
       void onPause();
   
       /**
        * 取消回调方法
        */
       void onCanceled();
   }
   
   
   ```

2. 下载功能 线程 AsyncTask

   DownTask.java

   ```java
   public class DownloadTask extends AsyncTask<String, Integer, Integer> {
       private static final int TYPE_SUCCESS = 0;
       private static final int TYPE_FAILED = 1;
       private static final int TYPE_PAUSED = 2;
       private static final int TYPE_CANCELED = 3;
   
       private DownloadListener downloadListener;
       private boolean isCancel;
       private boolean isPause;
       private int lastProgress;
   
       public DownloadTask(DownloadListener downloadListener) {
           this.downloadListener = downloadListener;
       }
   
       public void setPause(boolean pause) {
           isPause = pause;
       }
   
       public void setCancel(boolean cancel) {
           isCancel = cancel;
       }
   
       @Override
       protected Integer doInBackground(String... strings) {
           InputStream inputStream = null;
           RandomAccessFile savedFile = null;
           File file = null;
           try{
               // 已下载的文件长度
               long downloadedLength = 0;
               String downloadURL = strings[0];
               String[] subStrings = downloadURL.split("/");
               String fileName =subStrings[subStrings.length-1];
               String directory = MyApp.getContext().getApplicationContext().getFilesDir().getAbsolutePath();
               file = new File(directory+ fileName);
               Log.d("DS",directory);
               Log.d("DS",fileName);
               if (file.exists()){
                   boolean isDeleted = file.delete();
                   downloadedLength = file.length();
               }
               long contentLength = getContentLength(downloadURL);
               if (contentLength == 0){
                   // 下载失败
                   return TYPE_FAILED;
               }else if(contentLength == downloadedLength){
                   // 下载成功
                   return TYPE_SUCCESS;
               }
               // 网络操作
               OkHttpClient client = new OkHttpClient();
               Request request = new Request.Builder()
                       // 支持断点下载
                       .addHeader("RANGE", "bytes="+ downloadedLength +"-")
                       .url(downloadURL)
                       .build();
               Response response = client.newCall(request).execute();
               if (response.body() != null) {
                   inputStream = response.body().byteStream();
                   savedFile = new RandomAccessFile(file, "rw");
                   // 跳过已经下载的字节
                   savedFile.seek(downloadedLength);
                   byte[] buf = new byte[1024];
                   int total = 0;
                   int len;
                   // 正在下载
                   while ((len = inputStream.read(buf)) != -1){
                       if(isCancel)    return TYPE_CANCELED;
                       else if(isPause)    return TYPE_PAUSED;
                       else{
                           total += len;
                           savedFile.write(buf, 0, len);
                           // 计算已经下载的百分比
                           int progress = (int)((total+downloadedLength) * 100 / contentLength);
                           publishProgress(progress);
                       }
                   }
                   response.body().close();
                   return TYPE_SUCCESS;
               }
           }catch (Exception e){
               e.printStackTrace();
           }finally {
               if (inputStream != null) {
                   try {
                       inputStream.close();
                   } catch (IOException e) {
                       e.printStackTrace();
                   }
               }
               if (savedFile != null){
                   try {
                       savedFile.close();
                   } catch (IOException e) {
                       e.printStackTrace();
                   }
               }
               if (isCancel && file != null){
                   boolean delete = file.delete();
               }
           }
           return TYPE_FAILED;
       }
   
       private long getContentLength(String downloadURL) throws IOException {
           OkHttpClient client = new OkHttpClient();
           Request request = new Request.Builder()
                   .url(downloadURL)
                   .build();
           Response response = client.newCall(request).execute();
           if (response.body() != null && response.isSuccessful()){
               long contentLength = response.body().contentLength();
               response.body().close();
               return contentLength;
           }
           return 0;
       }
   
   
       @Override
       protected void onPostExecute(Integer integer) {
           switch (integer){
               case TYPE_SUCCESS:
                   downloadListener.onSuccess();
                   break;
               case TYPE_FAILED:
                   downloadListener.onFailed();
                   break;
               case TYPE_PAUSED:
                   downloadListener.onPause();
                   break;
               case TYPE_CANCELED:
                   downloadListener.onCanceled();
                   break;
               default:
                   break;
           }
       }
   
       @Override
       protected void onProgressUpdate(Integer... values) {
           int progress = values[0];
           if (progress > lastProgress){
               downloadListener.onProgress(progress);
               lastProgress = progress;
           }
       }
   
       @Override
       protected void onCancelled(Integer integer) {
           super.onCancelled(integer);
       }
   }
   
   ```

3. 服务DownloadService

   DownloadService.java

   ```java
   public class DownloadService extends Service {
       private DownloadTask downloadTask;
       private String downloadURL;
       private DownloadListener downloadListener = new DownloadListener() {
           @Override
           public void onProgress(int progress) {
               getNotificationManager().notify(1,getNotification("Downloading...", progress));
           }
   
           @Override
           public void onSuccess() {
               downloadTask = null;
               stopForeground(true);
               getNotificationManager().notify(1,getNotification("Download Success", -1));
               Toast.makeText(DownloadService.this,"Download Success", Toast.LENGTH_SHORT).show();
           }
   
           @Override
           public void onFailed() {
               downloadTask = null;
               stopForeground(true);
               getNotificationManager().notify(1,getNotification("Download Failed", -1));
               Toast.makeText(DownloadService.this,"Download Failed", Toast.LENGTH_SHORT).show();
           }
   
           @Override
           public void onPause() {
               downloadTask = null;
               Toast.makeText(DownloadService.this,"Download Paused", Toast.LENGTH_SHORT).show();
           }
   
           @Override
           public void onCanceled() {
               downloadTask = null;
               stopForeground(true);
               Toast.makeText(DownloadService.this,"Download Canceled", Toast.LENGTH_SHORT).show();
           }
       } ;
       private DownloadBinder mBinder = new DownloadBinder();
   
       private NotificationManager getNotificationManager(){
           return (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
       }
   
       private Notification getNotification(String title, int progress) {
           Intent intent = new Intent(this, ServiceActivity.class);
           PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent,0);
           NotificationCompat.Builder builder = new NotificationCompat.Builder(this,"default");
           builder.setSmallIcon(R.mipmap.ic_launcher_round);
           builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
           builder.setContentIntent(pendingIntent);
           builder.setContentTitle(title);
           if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
               builder.setPriority(Notification.PRIORITY_MAX);
           }
           if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
     //修改安卓8.1以上系统报错
           NotificationChannel notificationChannel = new NotificationChannel("xxx", "xxx",NotificationManager.IMPORTANCE_MIN);
           notificationChannel.enableLights(false);//如果使用中的设备支持通知灯，则说明此通知通道是否应显示灯
           notificationChannel.setShowBadge(false);//是否显示角标
           notificationChannel.setLockscreenVisibility(Notification.VISIBILITY_SECRET);
           notificationChannel.setImportance(NotificationManager.IMPORTANCE_HIGH);
           NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
               if (manager != null) {
                   manager.createNotificationChannel(notificationChannel);
               }
               builder.setChannelId("xxx");
     }
   
           if (progress >= 0){
               builder.setContentText(progress + "%");
               builder.setProgress(100, progress, false);
           }
           return builder.build();
       }
   
       public DownloadService() {
       }
   
       @Override
       public IBinder onBind(Intent intent) {
           // TODO: Return the communication channel to the service.
           return  mBinder;
       }
   
       public class DownloadBinder extends Binder {
           public void startDownload(String url){
               if (downloadTask == null){
                   downloadURL = url;
                   downloadTask = new DownloadTask(downloadListener);
                   downloadTask.execute(downloadURL);
                   startForeground(1, getNotification("Downloading...", 0));
                   Toast.makeText(DownloadService.this, "Downloading...",Toast.LENGTH_SHORT).show();
               }
           }
           public void pauseDownload(){
               if (downloadTask != null){
                   downloadTask.setPause(true);
               }
           }
   
           public void cancelDownload(){
               if (downloadTask != null){
                   downloadTask.setCancel(true);
               }
               if (downloadURL != null){
                   // 取消下载是需要删除文件，并关闭通知
                   String fileName = downloadURL.substring(downloadURL.lastIndexOf("/"));
                   String directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getPath();
                   File file = new File(directory+fileName);
                   if (file.exists()){
                       boolean isDeleted = file.delete();
                   }
                   getNotificationManager().cancel(1);
                   stopForeground(true);
                   Toast.makeText(DownloadService.this, "Canceled", Toast.LENGTH_SHORT).show();
               }
           }
       }
   }
   
   ```

4. 活动控制

ServiceActivity.java

```java
...
public class ServiceActivity extends AppCompatActivity {

    private Button btn_startDownload;
    private Button btn_pauseDownload;
    private Button btn_stopDownload;
    private TextView txt_tips3;

    private DownloadService.DownloadBinder downloadBinder;
    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (DownloadService.DownloadBinder)service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_service);
        btn_startDownload = findViewById(R.id.start_download);
        btn_pauseDownload = findViewById(R.id.pause_download);
        btn_stopDownload = findViewById(R.id.stop_download);

        txt_tips3 = findViewById(R.id.download_test);

        btn_startDownload.setOnClickListener(v->startDownload());
        btn_stopDownload.setOnClickListener(v-> downloadBinder.cancelDownload());
        btn_pauseDownload.setOnClickListener(v-> downloadBinder.pauseDownload());
    }

    private void startDownload() {
        Intent intent = new Intent(this, DownloadService.class);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            //android8.0以上通过startForegroundService启动service
            startForegroundService(intent);
        } else {
            startService(intent);
        }
        bindService(intent, serviceConnection, BIND_AUTO_CREATE);
        if (ContextCompat.checkSelfPermission(ServiceActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(ServiceActivity.this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},1);
        }
        if (downloadBinder == null)  return;
        String url = "https://down.qq.com/qqweb/PCQQ/PCQQ_EXE/PCQQ2020.exe";
        downloadBinder.startDownload(url);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (requestCode == 1){
            if (grantResults.length > 0 && grantResults[0] != PackageManager.PERMISSION_GRANTED){
                Toast.makeText(this, "拒绝权限将无法使用", Toast.LENGTH_SHORT).show();
                finish();
            }
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(serviceConnection);
    }
}


```

5. Manifest.xml

   ```xml
    <service
               android:name=".Service.DownloadService"
               android:enabled="true"
               android:exported="true"></service>
   ```

   