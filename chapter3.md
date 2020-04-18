<!-- toc -->

## 第三章 布局

###  android 控件可见属性

* visible: 可见且占据空间
* invisible：不可见但是占据空间
* gone ：不可见且不占据空间

使用`setVisibility()`方法设置， 可以传入

`View.VISIBLE`

 `View.INVIDIBLE` 

 `View.GONE`三种值

### ViewGroup

特殊的View,可以包含很多子View和ViewGroup

#### 线性布局LinearLayout

1. `android:orientation` 布局的方向 可选:horizontal vertical

2. `android:layout_gravity`: 控件在布局中的对齐方式

   `android:gravity `文字在控件中的对齐方式

3. `android:layout_weigth`

#### 相对布局RelativeLayout

可以使得控件出现在布局的任何位置

* 相对于父组件定位` layout_alignParent___ ="true"`

* 相对于其他控件布局 `layout_above = "@id/xxx"`

  `layout_to___Of = "@id/xxx"`

  引用其他控件的id

#### 帧布局FrameLayout

默认所有控件在左上角

`android:layout_gravity`: 上下左右  对齐的方向

### 引入布局

在其他布局中引入已经写好的布局

```xml
<include layout = "@layout/xxx">
```

隐藏系统自带的标题栏

```java
ActionBar ab = getSupportActionBar();
if(ab != null){
	ab.hide();
}
```

### 自定义控件

```java
public class NewLayout extends LinearLayout{
	public NewLayout(Context context, AttributeSet attrs){
		super(context, attrs);
		// 添加新布局
		LayoutInflater.from(context).inflate(R.layout.xxx, this);
        // TODO 添加有关动作
	}
}
```

`LayoutInflater.from(context)`得到一个`LayoutInflater`实例(布局填充器)

调用inflate方法动态加载布局, 第一个为布局文件id, 第二个是父布局

**可以在布局文件中直接使用**`NewLayout`

另外:

**获取View对象所在活动的方法**

```java
(Activity)getContext();
```



### ListView

#### 基本用法

1. 预先提供数据 `String data[] = {"A","B"}`

2. 建立Adapter  (Context context, sub_item_layout, data)

3. 找到listView实例

4. `setAdapter()`

   ```java
   // 2
   ArrayAdapter<String> adapter = new Adapter<String>(MainActivity.this,                                                   android.R.layout.simple_list_item_1,                                              data);
   // 3
   ListView lv =findViewById(xxx);
   // 4
   lv.setAdapter(adpter);
   ```

#### 定制界面

每个item_view 里有一张图片和一行文字

1. 数据类

```java
class Fruit{
    private String name;
    private int picId;
    
    Fruit(String name, picId){
        this.name = name;
        this.picId = picId;
    }
    
    public String getName(){
        return name;
    }
    
    public int getPicId(){
        return picId;
    }
}
```

2. 子类布局 略

3. 新建Adapter

   `resId`**用于指定当前某一项目布局文件(内部布局)**的id,

   `getView()`**每个划入屏幕范围的子项都会被调用**，在此加载其布局

   `convertView`**之前已经缓存的布局**

   ```java
   class FruitAdapter extends ArrayAdapter<Fruit>{
       // 子项布局文件的id
   	private int resId;
   	
   	public FruitAdapter(Context context, int textViewResourseId, List<Fruit> objects){
           super(context, textViewResourseId, objects);
           this.resid = textViewResourseId;
       }
       
       @override
       public View getView(int position, View convertView, ViewGroup viewgroup){
           Fruit fruit = getItem(position);
        // 为这个子项加载布局   
           View view = LayoutInflater.from(getContext()).inflate(resId,parent,false);
           ImageView imageview = view.getViewById(R.id.image);
           TextView textview = view.getViewById(R.id.text);
           imageview.setImageResource(fruit.getRPicId());
           textview.setText(fruit.getName());
           return view;
       }
   }
   ```

4. 掺入数据（MainActivity.java）

   ```java
   // 创建数据List
   List<Fruit> fruitList = new ArrayList<>();
   // 
   onCreate(){
       // ...
       // 填充List
       initFruit();
       FruitAdapter fruitadpater = new FruitAdapter(MainActivity.this, R.layout.xxx, fruitList);
       ListView lv = findViewById(xxx);
       lv.setAdapter(fruitadpater);
   }
   
   initFruit(){
       Fruit firit_1 = new Fruit("apple", R.drawable.xxx);
       fruitList.add(firit_1);
       // ...
   }
   ```

#### 优化之Adapter的内部类ViewHolder

`ViewHolder`可以用于缓存子项内部的实例，免得每次加载子项都要去`findViewById()`

```java
class FruitAdapter extends ArrayAdapter<Fruit>{
    // 子项布局文件的id
	private int resId;
	
	public FruitAdapter(Context context, int textViewResourseId, List<Fruit> objects){
        super(context, textViewResourseId, objects);
        this.resid = textViewResourseId;
    }
    
    @override
    public View getView(int position, View convertView, ViewGroup, viewgroup){
        Fruit fruit = getItem(position);
        View view ;
        
     	ViewHolder viewholder;
        // 如果之前没有缓存过布局
        if(convertView == null){
            view = LayoutInflater.from(getContext()).inflate(resId,parent,false);
            viewHolder = new ViewHolder();
            viewholder.textview = view.getViewById(R.id.text);
            viewholder.imageview = view.getViewById(R.id.image);
            // 将ViewHolder保存在view中
            view.saveTag(viewholder);
        }
        // 如果不为空，直接取来用
    	else{
            view = convertView；
            // 重新获取VIewholder
            viewholder = view.getTag();
        }
        viewholder.imageview.setImageResource(fruit.getRPicId());
        viewholder.textview.setText(fruit.getName());
        return view;
    }
    
    class ViewHolder{
        // 缓存实例
        TextView textview;
        ImageView imageview;
    }
}
```

### RecycleView 更推荐的滚动控件

`ViewHolder`内部要存放子控件**内控件的实例**，接收一个View对象，是**子布局的布局文件实例**

`FruitAdapter`构造函数完成数据的传入

`onCreateViewHolder`作用是创建`ViewHolder`实例并返回

`onBindViewHolder`对子项数据进行赋值, 根据`position`, 在每个子项滚动到屏幕内时执行

`getItemCount`简单返回子项数目即可

```java
import java.util.List;

public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder>{
    // 数据部分
    private List<Fruit> mFruitLists;

    static class ViewHolder extends RecyclerView.ViewHolder{
        IamegeView fruitImg;
        TextView fruirName;

        public ViewHolder(View view){
            super(view);
            fruitImg = (ImageView)view.findViewById(R.id.xxx);
            fruirName = (TextView)view.findViewById(R.id.yyy);
        }
    }

    public FruitAdapter(List<Fruit> fruitList){
        this.mFruitLists = fruitList;
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int ViewType){
        View view = LayoutInflater.from(getContext()).inflate(R.layout.zzz, parent, false);
        ViewHolder holder = new ViewHolder(view);
        return holder;
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position){
        Fruit fruit = mFruitLists.get(position);
        holder.fruitImg.setImageResource(fruit.getPicId());
        holder.fruirName.setText(fruit.getName());
    }

    @Override
    public int getItemCount(){
        return mFruitLists.size();
    }

}
```

MainActivity.java

`LayoutManager`用于指定RecyclerView的布局方式, `LinearLayoutManager`指的是线性布局, 默认纵向

```java
// onCreate()内
initFruit();
RecyclerView recyclerview = (RecyclerView)findViewById(R.id.aaa);
//
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
recycelerView.setLayoutManager(layoutManager);
//
FruitAdapter fa = new FruitAdapter(fruitList);
recyclerview.setAdapter(fa);
    
```

改为横向排列

```
layoutManager.setOrientation(LayoutManager.HONRIZONTAL);
```

* 对于有多种样式的控件，可以在同一个文件声明多种布局，**根据需要更改各个控件的可见性**，但仍然要获取所有控件的实例。实现(Vue中 v-if条件编译)的效果

* 当数据渲染后追加数据（通常是向一个List追加数据时），需要向adapter**声明追加, 需要刷新布局**

  参数：变更之后的长度int

  ```java
  adapter.notifyItemInserted(msgList.size());
  ```

  有需要可以使其滚到最后 

  参数：位置int

  ```java
  // msgRecyclerView是RecyclerView的实例
  msgRecyclerView。srcollToPosition(msgList.size() -1 )
  ```

  