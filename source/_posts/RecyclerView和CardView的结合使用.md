---
title: RecyclerView和CardView的结合使用
date: 2016-09-30 13:15:07
tags: [Android]
---



现在貌似还在使用ListView貌似就有点out了，最近在做项目的时候发现了还有RecyclerView和CardView结合使用的优雅做法，最后做出来效果：
![这里写图片描述](http://img.blog.csdn.net/20160930204657696)

----------
## 添加依赖 ##

```
 compile 'com.android.support:cardview-v7:23.1.1'
 compile 'com.android.support:design:24.0.0'
```

----------
## 每一个Item项CardView的布局 ##
RecyclerView的用法和ListView大同小异，但有一点不同，RecyclerView强制使用ViewHolder，这是个保存控件的类，需要自己写，这个后面再说。现在先来说说RecyclerView的item项每一个CardView的实现。
XML：

```
<android.support.v7.widget.CardView
    android:layout_width="380dp"
    android:layout_height="180dp"
    android:layout_marginTop="3dp"
    android:layout_marginLeft="3dp"
    android:layout_marginRight="3dp"
    android:layout_gravity="center_horizontal"
    app:cardCornerRadius="5dp"
    app:cardElevation="8dp"
    app:cardBackgroundColor="#FFFFE0"
    >
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <ImageView
            android:layout_width="160dp"
            android:layout_height="160dp"
            android:src="@mipmap/ic_launcher"
            android:id="@+id/pic_address"/>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_toRightOf="@id/pic_address"
            android:text="TEXTTEXTTEXT"
            android:textSize="20dp"
            android:id="@+id/des_bill"/>

        <TextView
            android:text="TextView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/date_bill"
            android:layout_alignParentBottom="true"
            android:layout_alignParentEnd="true"
            android:paddingBottom="5dp"
            android:paddingRight="5dp"/>
    </RelativeLayout>

</android.support.v7.widget.CardView>
```
注意`app:cardCornerRadius="5dp"
    app:cardElevation="8dp"
    app:cardBackgroundColor="#FFFFE0"`这几个CardView的属性分别表示卡片的圆角程度，卡片阴影，卡片背景颜色，写完后我们就可以开始实现Adapter了。



----------
## Adapter的实现 ##
上面说了，必须要实现ViewHolder，这里面要放所有item项的类。Adapter要继承自RecyclerView.Adapter<BillViewAdapter.MyHolder>括号里是自己实现的在Adapter的ViewHolder内部类.BillAdapter就是这个适配器类的名字。如下：

```
/**
     * 给控件绑定布局
     */
    public class MyHolder extends RecyclerView.ViewHolder
    {

        private TextView dateInfo;
        private ImageView picInfo;
        private TextView DesInfo;
        public MyHolder(View itemView)
        {
            super(itemView);
            dateInfo = (TextView) itemView.findViewById(R.id.date_bill);
            picInfo = (ImageView) itemView.findViewById(R.id.pic_address);
            DesInfo = (TextView) itemView.findViewById(R.id.des_bill);
        }
    }
```
我们还是要准备数据源，context上下文，和LayoutInflater布局填充器，在Adapter的构造方法里就要全部实现。

```
public BillViewAdapter(ArrayList<BillBean> beenList, Context context)
    {
        this.beanList = beenList;
        this.context = context;
        this.mInflater = LayoutInflater.from(context);
    }
```
接下来我们就要给Viewholder找当前布局文件了，不然去哪里绑定呢，在onCreateViewHolder方法里找到view

```
 public MyHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        View view = mInflater.inflate(R.layout.item_layout, parent,false);
        //返回到Holder
        return new MyHolder(view);
    }
```
这个方法返回的ViewHolder就会马上被onBindViewHolder接到，由于在ViewHolder里已经绑定了，所以在这里只需要把数据源的数据取出来，把控件setview。

```
 @Override
    public void onBindViewHolder(MyHolder holder, int position)
    {
        BillBean bean = beanList.get(position);
        holder.DesInfo.setText(bean.getDescripInfo());
        holder.picInfo.setImageResource(bean.getPicInfo());
        holder.dateInfo.setText(bean.getDateInfo());
    }
```
接下来` @Override
    public int getItemCount()
    {
        return beanList.size();
    }`也不要忘记了，这是ListView中也会有的方法。到这里Adapter就已经大功告成了。



----------
## 在Activity的使用 ##
在Activity中需要三个对象

```
 private RecyclerView recyclerView;
    private BillViewAdapter adapter;
    private RecyclerView.LayoutManager layoutManager;
```
第三个变量就是Listview没有的，这个是布局管理器，你可以通过它设置水平布局还是垂直布局。这样设置

```
recyclerView = (RecyclerView) findViewById(R.id.recycle_view);
        layoutManager = new LinearLayoutManager(this);
        recyclerView.setLayoutManager(layoutManager);
        adapter = new BillViewAdapter(beanList, this);
        recyclerView.setAdapter(adapter);
```
到这里我们的RecyclerView就设置完了，你还可以设置`setItemAnimator`，`addItemDecoration` 来设置动画和分界线。

