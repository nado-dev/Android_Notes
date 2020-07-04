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

启动服务：

```java
Intent intent = new Intent(this, MyService.class);
startService(intent);
```

关闭服务：

```java
Intent intent = new Intent(this, MyService.class);
stopService(intent);
```

