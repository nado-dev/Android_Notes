 <!-- toc -->

## 第九章 Android网络技术初步

### WebView

用于显示各种**网页** ( html + css + js )

* 布局

  ```xml
   <WebView
          android:id="@+id/webview"
          android:layout_width="match_parent"
          android:layout_height="match_parent"/>
  ```

* 代码

  ```java
   WebView webView = (WebView) findViewById(R.id.webview);
   // getSetting用于设置浏览器的属性
   // 允许js脚本
   webView.getSettings().setJavaScriptEnabled(true);
   // 作用是当要打开新网页时, 仍然在当前WebView中打开, 而不是通过系统浏览器
   webView.setWebViewClient(new WebViewClient());
   webView.loadUrl("http://android.aaronfang.fun");
  ```

  

### Android上的HTTP协议

#### 使用原生HttpURLConnection

1. 实例化一个URL类

2. 调用`openConnection()`方法, 得到一个连HTTP连接对象`HttpURLConnection`

3. 在连接对象`HttpURLConnection`中设置HTTP请求使用的方法`GET`, `POST`等

4. 设置超时, headers等

5. 调用连接对象的`getInputStream()`对象, 获得输入流`InputStream`, 对流可以进行读写

6. 关闭连接, 调用`disconnect()`关闭连接

   

   GET方法示例

   ```java
   public void loadJson(){
           new Thread(new Runnable() {
               @Override
               public void run() {
                   try {
                       // 定义URL
                       URL url = new URL("http://test2.aaronfang.fun/api/v1/postclass");
                       // 定义连接
                       HttpURLConnection connection =  (HttpURLConnection) url.openConnection();
                       // 设置超时时间
                       connection.setConnectTimeout(10000);
                       // 设置方法
                       connection.setRequestMethod("GET");
                       // 设置headers
                       connection.setRequestProperty("Content-Type", "application/json;charset=UTF-8");
                       connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
                       // 打开连接
                       connection.connect();
                       // 获取状态码
                       int resCode = connection.getResponseCode();
   
                       if (resCode == 200){
   //                        Map<String, List<String>> headerFields = connection.getHeaderFields();
   //                        Set<Map.Entry<String, List<String>>> entries = headerFields.entrySet();
   //                        for (Map.Entry<String, List<String>> entry : entries){
   //                            Log.d(TAG, entry.getKey()+" = " + entry.getValue());
   //                        }
   
                           InputStream inputStream = connection.getInputStream();
                           BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
   
                           String json = bufferedReader.readLine();
                           Log.d(TAG, "content" + " = "+ json);
   						// 更新UI
                           updateUI();
                       }
   
                   } catch (Exception e) {
                       e.printStackTrace();
                       connect.disconnect();
                   }
   
               }
           }).start();
       }
   ```

   **网络有关的操作需要在子线程内进行**

   **而UI的更新不允许在子线程中运行, 需要使用特定的方法将其切换回主线程**

   ```java
   private void updateUI() {
           runOnUiThread(new Runnable() {
               @Override
               public void run() {
                   tv_confirm.setText("load! ");
               }
           });
       }
   ```



POST方法示例 (部分):

```java
connection.setRequestmethod("POST");
// 获取连接的输出流
DataOutputStream out = new DataOutputStream(connection.getOutputStream);
// 向输出流写入POST携带的信息 数据之间使用 & 隔开
out.writeBytes("userName=admin&password=1234");
```



****

UI控件介绍:`ScrollView`

滚动视图, 当一个页面内容太多需要滚动显示时使用

可以嵌套其他控件

****



#### 开源框架OkHttp

依赖

```
implementation 'com.squareup.okhttp3:okhttp:3.14.1'
```



1. 实例化客户端`OkHttpClient`
2. 实例化一个请求对象`Request`, 通过`Request.Builder()`的一系列连缀方法丰富这个请求对象， 最后调用build()提交
3. 向客户端提交这个请求对象`newCall(request).execute()`  返回一个`Request`对象, 里面含有返回

`GET`方法示例:

```java
	private void loadjson() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 实例化OkHttpClient ，实例化客户端
                OkHttpClient client = new OkHttpClient();
                // 实例化Request对象， 通过Request.Builder()的一系列连缀方法丰富这个请求对象， 最后调用build()提交
                Request request = new Request.Builder()
                        .url("http://test2.aaronfang.fun/api/v1/postclass")
                        .build();
                try {
                    Response response = client.newCall(request).execute();
                    int recode = response.code();
                    if (recode == 200){
                        String res = response.body() != null ? response.body().string() : "empty body";
                        Log.d("OKHTTP", "body"+res);
                        UIUpdate(res);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();

    }

 private void UIUpdate(String res) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
               tv_json.setText(res);
            }
        });
    }
```

`POST`示例:

`POST`方法需要提前构造请求体`body` 通过`RequestBody`类进行构建

`FormBody.Builder`构建表单

```java
RequestBody requestBody = new FormBody.Builder()
.add("username", "admin")
.add("password", "123")
.build();
```

再再构造请求对象时调用`post(RequestBody requestBody)`方法

```java
 Request request = new Request.Builder()
                        .url("http://xxx")
     					.post(requestBody)
                        .build();
```

### JSON格式数据解析

#### 官方的JSONObject

待解析的数据

```json
{
    "msg":"获取成功",
    "data":{
        "list":[
            {
                "id":1,
                "classname":"时间线"
            },
            {
                "id":2,
                "classname":"综合1"
            },
            {
                "id":3,
                "classname":"速报2"
            },
            {
                "id":4,
                "classname":"欢乐恶搞"
            },
            {
                "id":5,
                "classname":"跑团"
            },
            {
                "id":6,
                "classname":"圈内"
            }
        ]
    }
}
```

使用`get`方法取出数据

```java
private void parseJSONWithJSONObject(String res) {
        try {
                JSONObject jsonObject = new JSONObject(res);
                String status = jsonObject.getString("msg");
                String data = jsonObject.getString("data");

                Log.d("OkHttp", "msg="+status);
                Log.d("OkHttp", "data="+data);
;            
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```

JSON对象 `JSONObject`

JSON数组 `JSONArray`

#### GSON

#### GsonFormat插件生成实体类

* 安装GsonFormat实体类

  `Files -> Settings -> Plugins  -> search GsonFormat  `

  快捷键打开`Alt + Insert -> GsonFormat `   或  `Alt + S`

* 生成实体类

  `GetTextItem.java`

  ```java
  package com.example.androidnetworkdemo.domain;
  
  import java.util.List;
  
  public class GetTextItem {
      /**
       * msg : 获取成功
       * data : {"list":[{"id":1,"classname":"时间线"},{"id":2,"classname":"综合1"},{"id":3,"classname":"速报2"},{"id":4,"classname":"欢乐恶搞"},{"id":5,"classname":"跑团"},{"id":6,"classname":"圈内"}]}
       */
  
      private String msg;
      private DataBean data;
  
      public String getMsg() {
          return msg;
      }
  
      public void setMsg(String msg) {
          this.msg = msg;
      }
  
      public DataBean getData() {
          return data;
      }
  
      public void setData(DataBean data) {
          this.data = data;
      }
  
      public static class DataBean {
          private List<ListBean> list;
  
          public List<ListBean> getList() {
              return list;
          }
  
          public void setList(List<ListBean> list) {
              this.list = list;
          }
  
          public static class ListBean {
              /**
               * id : 1
               * classname : 时间线
               */
  
              private int id;
              private String classname;
  
              public int getId() {
                  return id;
              }
  
              public void setId(int id) {
                  this.id = id;
              }
  
              public String getClassname() {
                  return classname;
              }
  
              public void setClassname(String classname) {
                  this.classname = classname;
              }
          }
      }
  }
  
  ```

  解析:

  [源数据](#官方的JSONObject)

  data项抽象为DataBean类
  data中的list对象抽象为listBean

	```java
	private void parseJSONWithGson(String res) {
      Gson gson = new Gson();
        // 得到JSON对象
        GetTextItem getTextItem = gson.fromJson(res, GetTextItem.class);
        String status = getTextItem.getMsg();
  
      GetTextItem.DataBean dataBean = getTextItem.getData();
        List<GetTextItem.DataBean.ListBean> list = dataBean.getList();
  
      for (GetTextItem.DataBean.ListBean listBean : list){
            int id = listBean.getId();
            String classname = listBean.getClassname();
  
          String info_d = "id = "+ id +"classname" + classname+ "\n";
            info = info.concat(info_d);
        }
    }
  ```

### 最佳实践 HTTP工具类的封装

#### HttpRequest简单封装

声明接口来实现回调

如果在``sendHttpRequestByGetMethod()``内开启子线程进行网络操作。 服务器的响应数据是无法返回的， 该方法在服务器还没来得及相应的时候就已经结束了。

这时就要使用Java的回调机制，先定义一个接口， 定义执行成功或失败的方法

```java
public interface HttpCallbackListener {
    void onFinish(String response);

    void onError(Exception e);
}
```

然后在`sendHttpRequestByGetMethod()`方法中传入实现这个接口的类的对象（即要求实现这个函数时先重写对应的方法，换种说法，`sendHttpRequestByGetMethod()`方法定义的成功或失败或其他事件发生的条件和时机等， 在调用时要求实现这些方法的具体操作， 以及接收这些操作的返回）

```java
    public static void sendHttpRequestByGetMethod(final String address, final HttpCallbackListener listener){
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    URL url = new URL(address);
                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                    // 设置超时时间
                    connection.setConnectTimeout(10000);
                    // 设置方法
                    connection.setRequestMethod("GET");
                    // 设置headers

                    connection.connect();
                    InputStream in = connection.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null) {
                        response.append(line);
                    }
                    if (null != listener) {
                        // onFinish
                        listener.onFinish(response.toString());
                    }

                } catch (Exception e) {
                    if (listener != null) {
                        listener.onError(e);
                    }
                    e.printStackTrace();
                }
            }
        }).start();
    }

```

调用

```java
 btn_httputil.setOnClickListener((v)->{
   HttpUtil.sendHttpRequestByGetMethod("http://test2.aaronfang.fun/api/v1/postclass",
                    new HttpCallbackListener() {
                        @Override
                        public void onFinish(String response) {
                            updateUI();
                            Log.d("httputils", "success");
                        }

                        @Override
                        public void onError(Exception e) {

                        }
                    });
        }

```



#### OkHttp简单封装

HttpUtil

`okhttp3.Callback`是OkHttp库中自带的一个回调接口, 在`client`调用`newCall()`后没有调用`execute()`方法, 调用的是`enqueue()`, 因为该方法的内部已经开启了一个线程, 在子线程在进行HTTP请求, 并将最后的请求结果回掉到`okhttp3.Callback`中

```java
 public static void sendOkHttpRequestByGetMethod(String address, okhttp3.Callback callback){
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(address)
                .build();
        client.newCall(request).enqueue(callback);
    }
```

调用

```java
 btn_okhttpTest.setOnClickListener((view)->{
 HttpUtil.sendOkHttpRequestByGetMethod("http://test2.aaronfang.fun/api/v1/postclass", new okhttp3.Callback() {
                @Override
                public void onFailure(Call call, IOException e) {

                }

                @Override
                public void onResponse(Call call, Response response) throws IOException {
                    String responseData = response.body().string();
                    Log.d("Okhttp test", responseData);
                    updateUI();
                }
            });
        });

```

