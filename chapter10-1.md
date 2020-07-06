<!-- toc -->

## 第十章-1 Android 多线程编程

### :one: Android 多线程编程

#### 基本用法

1. 一般Java多线程语法：继承Thread类，重写run方法；创建实例，调用start方法

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        Log.d("Thread", "Hello World");
    }
    
    public void test(){
        MyThread myThread = new MyThread();
        myThread.start();
    }
}
```

2. 实现Runnable接口

```java
public class MyThread implements Runnable {
    @Override
    public void run() {
        Log.d("Thread", "Hello World");
    }

    public void test(){
        MyThread myThread = new MyThread();
        new Thread(myThread).start();
    }
}
```

​	更常用的写法

```java
 public void test(){
       new Thread(new Runnable() {
           @Override
           public void run() {
               
           }
       });
   }
或
 public void test(){
       new Thread(() -> {
           // 逻辑
       });
   }
```

### :two: 在子线程中更新UI

​	Android的UI是线程不安全的。**要更新UI，必须要在主线程中进行**。

​	一般的情况是，要在子线程中进行耗时操作，依据操作的结果更新相应的UI控件。

​	对于这种情况，Android提供了一套异步消息处理机制。

ThreadActivity.java

```java
public class ThreadActivity extends AppCompatActivity {
    TextView txt_hw;
    Button btn_changeUI;
    public static final int UPDATE_TEXT = 1;
    private Handler handler = new Handler(){
        @Override
        public void handleMessage(@NonNull Message msg) {
            if (msg.what == UPDATE_TEXT) {// 进行UI操作
                txt_hw.setText("Text changed");
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_thread);
        txt_hw = findViewById(R.id.txt_hw);
        btn_changeUI = findViewById(R.id.btn_changeUI);

        btn_changeUI.setOnClickListener(v->{
            new Thread(()->{
               Message message = new Message();
               message.what = UPDATE_TEXT;
               handler.sendMessage(message);
            }).start();
        });

    }
}
```

1. 将更新UI这一事件用UPDATE_TEXT = 1指代

2. 新增一个handler对象，重写父类的`handleMessage()`方法。如果Message实例的what为UPDATE_TEXT ，进行具体的UI处理。

3. 由于不能再子线程中进行UI操作，故在子线程中发出修改UI的**消息**，new出Message的实例，将what设置为UPDATE_TEXT ，再调用`sendMessage()`方法发送消息。

   > 更推荐使用`Message.obtain()` 方法获取Message的实例。不建议直接new Message，Message内部保存了一个缓存的消息池，我们可以用obtain从缓存池获得一个消息，Message使用完后系统会调用recycle回收，如果自己new很多Message，每次使用完后系统放入缓存池，会占用很多内存的。 

4. Handler收到消息后在**主线程内执行`handleMessage`方法实现更新UI**

### :three: 异步消息处理机制原理解析

> 参考 https://www.jianshu.com/p/e635c3f851ea

四个组成部分：Message、Looper、MessageQueue、Handler

1. Message

   线程之间传递的消息，它可以在内部携带少量的信息（FLAG类的消息居多）,用于在不同的线程之间交换数据。有what(int)字段, arg1和arg2字段（int，更低的开销）以及Object类型的obj字段。

2. Handler

   消息处理者，发送消息一般通过`sendMessage(Message message)`方法，发出消息通过一系列的处理之后，最终会传到Handler`handleMessage(@NonNull Message msg)`中。

3. MessageQueue

   消息队列。它用于存放所有通过Handler发送的消息。这部分消息会一直存在与消息队列中。**每个线程只有一个MessageQueue对象。**

4. Looper

   是每个线程中的Message的管家，调用Looper的`loop()`方法之后，进入到一个午线循环中，每当发现线程关联的Message存在一条消息，就会将他取出，并传递到`Handler的handlerMessage()`中。每个线程中也只有一个Looper对象。通过Looper.perpare获取它的实例。



全流程：

主线程创建Handler对象，重写`handleMessage(Message msg)`方法

--> 子线程需要UI操作时创建Message对象，通过handler将这条消息发送出去。

--> 这条消息会发送到MessageQueue中的队列中等待处理。

--> Loopper会一直尝试从MessageQueue中取出消息

--> 分发给Handler的`handleMessage(@NonNull Message msg)`



### :four: AsyncTask

​	一套封装易用的异步消息处理机制

​	是一个抽象类，需要子类继承使用三个泛型参数可以指定

​	public abstract class AsyncTask<Params, Progress, Result>

* Params：在执行AsyncTask时**需要传入的参数**，在后台任务中可用。
* Progress：后台执行任务时，如果需要在界面上显示当前的进度，则使用在这里指定的泛型作为**进度单位**。
* Result：如果结果需要返回，如果结果需要返回，这里使用的泛型作为**返回值类型**。

需要重写的一些方法：

* onPreExecute()

  在后台任务**开始执行之前调用**，进行初始化操作。如界面操作显示进度条对话框等。

* :star: Result doInBackground(Params...)

  这个方法中的所有代码都在**子线程**中运行，应该在这里执行所有耗时操作。

  一旦完成调用return语句来将任务的执行结果返回，若Result指定为Void，表示可以不用返回执行结果。

  **这个方法不可以执行UI操作**，如果要显示进度等UI操作，可以调用`publishProgress(Progress...)`方法。

* onPublishProgress(Progress...)

  **当后台调用了publishProgress(Progress...)方法后**，onPublishProgress(Progress...)很快被调用，该方法中携带的参数是后台任务中传过来的。

  在这里可以进行UI操作，通过参数可以对界面进行更新。

* onPostResult(Result)

  **后台任务执行完毕通过return语句返回时**，很快被调用。return语句传来的参数，可以利用返回来的数据进行一些UI操作

  Demo

  ```java
  class DownloadTask extends AsyncTask<Void, Integer, Boolean>{
          @Override
          protected void onPreExecute() {
              // 显示进度框
              progressDialog.show();
          }
          
          @Override
          protected void onPostExecute(Boolean aBoolean) {
              progressDialog.dismiss();
              if (aBoolean){
                  // 提示成功
              }else{
                  // 提示失败
              }
          }
  
          @Override
          protected void onProgressUpdate(Integer... values) {
              // 更新下载进度
              progressDialog.setMessage("info" + Arrays.toString(values));
          }
  
          @Override
          protected Boolean doInBackground(Void... voids) {
              try{
                  while (true){
                      int downloadPercent = doDwnload();
                      publishProgress(downloadPercent);
                      if (downloadPercent >= 100) break;
                  }
              }catch (Exception e){
                  e.printStackTrace();
                  return false;
              }
              return true;
          }
      }
  ```

  如需调用

  ```java
  new DownLoadTask().execute();
  ```

  
  
  关于[RandomAccessFile](https://blog.csdn.net/dimudan2015/article/details/81910690?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

