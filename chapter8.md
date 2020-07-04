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

###  :two: 摄像头

Demo

1. 创建File对象，并把它的目录设置为**应用关联缓存目录**，指SDcard中专门用于存放当前应用缓存的地方，通过调用`getExternalCacheDir()`获取。

   > SDK 24 （Android 6.0 ）后存放文件进入SDcard被认为是一个危险权限，需要申请运行时权限，存放进应用关联缓存目录可以避免这步

2. 如果时SDK 24之前的，可以直接将File文件转为Uri对象`Uri.fromFile(File file)`，这样的Uri表示着真实路径

   如在SDK 24后（含24），需要调用FileProvider的`getUriForFile(Context context, String authority, File file)`, 第二个参数可以是唯一的字符串。

   > FileProvider可以选择性地提供封装过的Uri，提高安全性

3. Intent启动相机，将Action指定为`"android.media.action.IMAGE_CAPTURE"`

   在向Intent传入保存的地址 `intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);`

4. 启动活动，因为需要回调所以要使用`startActivityForResult(Intent intent)`方法
5. 在回调方法onActivityResult中，将文件解析为Bitmap，展示到ImageView中

```java
public class CamActivity extends AppCompatActivity {
    private ImageView imageView;
    private Button button;
    private Uri imageUri;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_cam);

        imageView = findViewById(R.id.imageView_pic);
        button    = findViewById(R.id.button_pic);

        button.setOnClickListener(v->{
            File outputImage = new File(getExternalCacheDir(), "out_image.jpg");
            try{
                if (outputImage.exists()){
                    outputImage.delete();
                }
                outputImage.createNewFile();
            }catch (Exception e){
                e.printStackTrace();
            }
            if (Build.VERSION.SDK_INT >=24){
                imageUri = FileProvider.getUriForFile(CamActivity.this, "com.example.sqlite_demo.fileprovider",outputImage);
            }
            else{
                imageUri = Uri.fromFile(outputImage);
            }
            // 启动相机
            Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
            intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
            startActivityForResult(intent,1);
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if (requestCode == 1) {
            if (resultCode == RESULT_OK) {
                try {
                    Bitmap bitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
                    imageView.setImageBitmap(bitmap);
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

5. 设置manifest.xml provider

manifest.xml

```xml
<provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.example.sqlite_demo.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"/>
</provider>
```

android:authorities 的值与FileProvider.getUriForFile的第二个参数相同

6. 设置共享对象

<meta-data> 中指定Uri的共享路径，新建res/xml文件夹，新建文件file_paths.xml

file_paths.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="" />
</paths>
```

name随意

path的值表示需要贡献的路径（范围）设置为空值表示共享整个SD卡（不推荐），也可以只共享这张图片的路径

### :three: 相册

GalleryActivity.java

1. 申请运行时权限READ_EXTERNAL_STORAGE

2. Intent 获取本地图片

   ```java
   Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
   intent.setType("image/*");
   tartActivityForResult(intent,CHOOSE_PHOTO);
   ```

   > `Intent.ACTION_GET_CONTENT`获取的是所有本地图片， `Intent.ACTION_PICK`获取的是相册中的图片。
   >
   > ```java
   > Intent intent = new Intent(Intent.ACTION_PICK,android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
   > intent.setType("image/*");
   > startActivityForResult(intent, REQUEST_CODE_ALBUM);
   > 
   > ```
   >
   > Intent.ACTION_PICK返回的uri格式只有一种：比如
   > uri=content://media/external/images/media/34；
   >
   > 而Intent.ACTION_GET_CONTENT返回的uri格式：
   >
   > 在Android版本4.4以下同Intent.ACTION_PICK返回的一 样，
   >
   > Android版本4.4以上则返回多种格式：如
   > uri=content://com.android.providers.media.documents/document/image%3A245743，
   > uri=file:///storage/emulated/0/temp_photo.jpg，
   > uri=content://media/external/images/media/193968
   > …
   >
   > Intent.ACTION_GET_CONTENT必须设置setType("image/*")表示返回的数据类型，否则会报
   > android.content.ActivityNotFoundException异常。
   >
   > [参考 CSDN 博主「CoderCF」](https://blog.csdn.net/chengfu116/article/details/74923161)
   
3. 对于KitKat以上的版本，由于返回的格式不同，要做不同的适配

   对于Document类型：

   * authority为media格式：如content://media/external/images/media/193968，需要截取出最后的id，构造出selection（即where语句），把MediaStore.Images.Media.EXTERNAL_CONTENT_URI与selection结合进行查找
   * download格式需要构造content://downloads/public_downloads + docId的形式的Uri进行查找

   对于content类型uri可以直接进行查找

   对于file类型getPath()即可，无需查找

4. 将图片路径转为Bitmap，绘制即可

   > Android 10(API >= 29) 禁止了通过路径来访问其他程序的资源，可以暂时停用这个feature以实现下面的代码。
   >
   > 在manifest.xml中，设置requestLegacyExternalStorage="true"
   >
   > ```xml
   > <manifest ... >
   > <!-- This attribute is "false" by default on apps targeting Android 10 or higher. -->
   > 	<application android:requestLegacyExternalStorage="true" ... >
   >         ...
   > 	</application>
   > </manifest>
   > ```
   >
   > [了解更多](https://developer.android.google.cn/training/data-storage/files/external-scoped)

```java
public class GalleryActivity extends AppCompatActivity {
    private static final int CHOOSE_PHOTO = 2;
    private Button btn_gly;
    private ImageView img_gly;

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_gallery);
        btn_gly = findViewById(R.id.button_gly);
        img_gly = findViewById(R.id.imageView_gly);

        btn_gly.setOnClickListener(v->{
            int currentPermission = ContextCompat.checkSelfPermission(GalleryActivity.this, Manifest.permission.READ_EXTERNAL_STORAGE);
            int grantResult = PackageManager.PERMISSION_GRANTED;
            if (currentPermission != grantResult){
                ActivityCompat.requestPermissions(GalleryActivity.this, new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},1);
            }
            else{
                openGallery();
            }
        });
    }

    private void openGallery() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("image/*");
        startActivityForResult(intent,CHOOSE_PHOTO);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (requestCode == 1){
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                openGallery();
            }
            else{
                Toast.makeText(this, "permission grant Failed",Toast.LENGTH_SHORT).show();
            }
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == CHOOSE_PHOTO){
            if (resultCode == RESULT_OK){
                if (Build.VERSION.SDK_INT >= 19){
                    handleImageOnKitKat(data);
                }
                else{
                    handleImageBeforeKitKat(data);
                }
            }
        }
    }

    @TargetApi(19)
    private void handleImageOnKitKat(Intent data) {
        String imagePath = null;
        Uri uri = data.getData();
        // 如果是Document类型的Uri
        if (DocumentsContract.isDocumentUri(this, uri)){
            String docId = DocumentsContract.getDocumentId(uri);
            if (uri != null && "com.android.providers.media.documents".equals(uri.getAuthority())) {
                String id = docId.split(":")[1];
                Log.d("ID",id);
                String selection = MediaStore.Images.Media._ID + "=" + id;
                imagePath = getImagePath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, selection);
            }
            else if(uri != null && "com.android.providers.downloads.documents".equals(uri.getAuthority())){
                Uri contentUri = ContentUris.withAppendedId(Uri.parse("content://downloads/public_downloads"), Long.valueOf(docId));
                imagePath = getImagePath(contentUri, null);
            }
        }
        else if (uri != null &&"content".equalsIgnoreCase(uri.getScheme())){
            imagePath = getImagePath(uri, null);
        }
        else if (uri != null && "file".equalsIgnoreCase(uri.getScheme())){
            imagePath = uri.getPath();
        }
        displayPic(imagePath);
    }

    private void displayPic(String imagePath) {
        if (imagePath != null){
            Bitmap bitmap = BitmapFactory.decodeFile(imagePath);
            img_gly.setImageBitmap(bitmap);
        }
        else{
            Toast.makeText(this, "failed", Toast.LENGTH_SHORT).show();
        }
    }

    private void handleImageBeforeKitKat(Intent data) {
        Uri uri = data.getData();
        String imagePath = getImagePath(uri, null);
        displayPic(imagePath);
    }
```



