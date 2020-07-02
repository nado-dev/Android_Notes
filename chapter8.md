<!-- toc -->

## 第八章 简单多媒体API及其运用

### :one: 通知Notification

创建通知的步骤：

1. 通过Context的`getSystemService(Context.NOTIFICATION_SERVICE)`来获取NotificationManager实例

2. 通过NotificationCompat类的Builder来构建通知。

3. NotificationManager的实例调用notify创建通知。

   ```java
    NotificationManager notificationManager = (NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE);
                   Notification notificationOne =
                           new NotificationCompat.Builder(MainActivity.this,"default")
                                   .setContentTitle("test")
                                   .setContentText("texttexttexttexttexttexttexttexttexttexttexttexttext")
                                   .setWhen(System.currentTimeMillis())
                                   .setSmallIcon(R.drawable.ic_launcher_background)
                                   .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher_foreground))
                                   .build();
                   if (notificationManager != null) {
                       notificationManager.notify(1,notificationOne);
                   }
   ```

   setContentTitle ：标题

   setWhen：指定通知被创建的时间

   setSmallIcon：小图

   setLargeIcon：大图

**为通知赋予点击操作 PendingIntent**

PendingIntent类似于Intent，Intent倾向于立即执行动作，PendingIntent倾向于延迟执行

PendingIntent有getActivity(), getBroadcast(), getService()方法等，参数一样

(Context context, 0, Intent intent, 0)， 第四个参数见文档

![image-20200702154014495](.\chapter8.assets\image-20200702154014495.png)

改进后

```java
Intent intent = new Intent(MainActivity.this,LitePalDemoActivity.class);
                PendingIntent pendingIntent = PendingIntent.getActivity(MainActivity.this,0,intent,0);
// 以上创建通知的代码
// Builder中增加连缀.setContentIntent(pendingIntent)
```

**取消通知**

1. 自动取消

    Builder中增加连缀  `.setAutoCancel(true)`

2. 手动取消

   在适合取消的地方

   ```java
    NotificationManager notificationManager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
           if (notificationManager != null) {
               notificationManager.cancel(1);
           }
   ```

   cancel中传入的值是定义通知时的编号

**其他API**

1. 播放声音

    Builder中增加连缀  传入声音文件的Uri

   ` .setSound(Uri.fromFile(newFile("system/media/audio/ringtones/Luna.ogg")))`

2. 震动

   Builder中增加连缀

   `.setVibrate(new long[]{0,1000,1000,1000})`

   > 数组的下标为偶数代表震动的毫秒数，奇数代表停止的毫秒数

   需要有关权限

   ```xml
   <uses-permission android:name="android.permisson.VIBRATE"
   ```

3. 默认效果

   Builder中增加连缀

   `.setDefaults(NotificationCompat.DEFAULT_ALL)`

**高级功能**

1. 通过连缀的`setStyle()`方法实现更多效果

   长文本, 传入NotificationCompat.BigTextStyle()的实例 ,调用bigText方法传入字符串

   ```java
    .setStyle(new NotificationCompat.BigTextStyle().bigText("" +
                                           "test需要有关权限\n" +
                                           "\n"))
   ```

   大图模式，传入NotificationCompat.BigTextStyle()的实例，调用bigPicture方法传入Bitmap

   ```java
    .setStyle(new NotificationCompat.BigPictureStyle()                                    .bigPicture(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher)))
   ```

2. 优先级

   setPriority(), 参数在Notification中的5个常量中选择，从高到低

![image-20200702163514434](.\chapter8.assets\image-20200702163514434.png)

PRIORITY_DEFAULT: 相当于不设置

PRIORITY_HIGH: 弹出横幅

PRIORITY_MIN: 只在特定的场合出现，如下拉状态栏时

PRIORITY_LOW: 相对不重要，顺序较低

PRIORITY_MAX: 最重要

###  :two: 摄像头和相册

