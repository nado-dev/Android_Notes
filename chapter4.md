<!-- toc -->

## 第四章 fragment碎片

碎片Fragment是一段可以嵌入活动中的UI片段

**可以理解成是迷你的活动**

### 基本用法 显式添加碎片

1. 创建碎片的布局 （LeftFragment.xml RightFragment.xml）

2. 创建相应的类 (leftFragement为例)

   继承Fragment类

   重写onCreateView方法, 获取view对象

   ```java
   public class LeftFragment extends Fragment{
       @override
       public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInatanceState){
           View view = inflater.inflate(R.layout.LeftFragment, container, false);
           return view
       }
   }
   ```

3. 创建容纳布局的大布局文件

   ```xml
   <LinearLayout xxx>
       <fragment
            android:id="@+id/xxx"
            android:name="[指定布局添加碎片的类名的包路径]"
       .../>
   </LinearLayout>
   ```

4. MainAtivity中默认基本载入3中布局即可

### 动态加载碎片

1. 在大的布局文件中使用`FrameLayout`占位

   ```xml
   <LinearLayout xxx>
       <fragment
            android:id="@+id/xxx"
            android:name="[指定布局添加碎片的类名的包路径]"
       .../>
       <FrameLayout
        	android:id="@+id/yyy"
                    />
   </LinearLayout>
   ```

2. 事先完成 显式添加碎片中的1, 2两个步骤

3. 在`MainActivity`中添加如下函数以完成替换功能

   

   ```java
   private void repalceFragment(Fragment fragment){
       // 1.获取fragmentManger, 在活动在直接通过 getSupportFragmentManager()得到
       FragmentManager fragmentManger = getSupportFragmentManager();
       // 2.开启一个事务
       FragmentTransaction transaction = fragmentManager.beginTransaction();
       // 3.替换传入 (被替换的容器id, 新的碎片实例)
       transaction.replace(R.id.yyy, fragment);
       // 4.提交事务
       transaction.commit();
   }
   ```

   调用

   ```java
   replaceFragment(new RightFragment());
   ```

   ### 模拟返回栈

   ```java
   private void repalceFragment(Fragment fragment){
       // 1.获取fragmentManger, 在活动在直接通过 getSupportFragmentManager()得到
       // FragmentManager fragmentManger = getSupportFragmentManager();
       // 2.开启一个事务
       // FragmentTransaction transaction = fragmentManager.beginTransaction();
       // 3.替换传入 (被替换的容器id, 新的碎片实例)
       // transaction.replace(R.id.yyy, fragment);
       // 4.加入返回栈
       transaction.addToBackStack(null);
       // 5.提交事务
       // transaction.commit();
   }
   ```

   

### 碎片与活动之间的通信

#### 活动中获取碎片实例

MainActivity.java

```java
SampleFragment sampleFragment = (SampleFragment)getSupportFragmentmanager()
    .findFragmentByid(R.id.zzz);
```

#### 碎片中获取实例

SampleFragment.java

```java
MainActivity activity = (MainActivity) getActivity();
```

### 生命周期

* onAttach() 单碎片与活动建立关联时调用
* onCraeteView() 为碎片创建视图 加载布局时调用 
* onActivityCreated() 确保与碎片相关联的活动创建完毕时调用
* onDestoryView() 当与碎片关联的试图被移除时调用
* onDetch() 单碎片与试图解除关联时调用

### 适配大屏

在res目录下新建``layout-large`目录, 里面的布局会在被认为是大盘的设备上加载.

自定义:最小宽度限定符`layout-sw[]dp`

sw = samllest width , []填数字 