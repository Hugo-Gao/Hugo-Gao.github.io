---
title: AlertDialog的几种用法
date: 2016-09-14 13:10:55
tags: [Android]
---



AlertDialog就是在屏幕上出现一个对话框，并且要获取当前Activity的焦点，也就是说只能在对话框中进行操作。

----------
## 单调的确认对话框 ##
这是最简单的一种dialog形式，可以在对话框里加入图片，标题，呢容，以及两个按钮。dialog都是用`AlertDialog.Builder builder = new AlertDialog.Builder(this); ` 来进行创建的，需要在builder中定义好属性后才能创建dialog对象。代码很简单，通过dialog的几个方法就可以一一设置了。如果你不想用户通过点击对话框外的地方退出对话框，还可以调用setCancelable方法。

```
 private void showDialog()
    {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setIcon(R.mipmap.ic_launcher);
        builder.setTitle("确认对话框");
        builder.setMessage("确认对话框中的内容");//提示内容
        builder.setPositiveButton("确定啊", new DialogInterface.OnClickListener()
        {
            @Override
            public void onClick(DialogInterface dialog, int which)
            {
                Toast.makeText(MainActivity.this, "点击了确定按钮", Toast.LENGTH_SHORT).show();
            }
        });
        builder.setNegativeButton("取消啊", new DialogInterface.OnClickListener()
        {
            @Override
            public void onClick(DialogInterface dialog, int which)
            {
                Toast.makeText(MainActivity.this, "点击了取消按钮", Toast.LENGTH_SHORT).show();
            }
        });
        AlertDialog dialog = builder.create();
        dialog.show();
    }
```
最后要记得调用show方法，dialog才能出现。效果如下：
![这里写图片描述](http://img.blog.csdn.net/20160914145701023)

----------
## 单多选对话框的用法 ##
单选对话框其实很简单，其他都是相同的，只需要多调用builder的这个方法

```
 builder.setSingleChoiceItems(single_list, 0, new DialogInterface.OnClickListener()
        {
            @Override
            public void onClick(DialogInterface dialog, int which)
            {
                String sex = single_list[which];
                Toast.makeText(MainActivity.this, "你选择的是"+sex, Toast.LENGTH_SHORT).show();
            }
        });
```
这`setSingleChoiceItems` 方法的第一个参数就是待选数据，你需要用一个字符串数组把他保存起来`private String[] single_list={"男","女","女博士"};` 第三个参数就是点击事件了，在onclick方法里第二个参数就是数组下标。很简单。
![这里写图片描述](http://img.blog.csdn.net/20160914151112996)
多选对话框只需要改动一下`setSingleChoiceItems` 为 `setMultiChoiceItems` 第一个参数依然传数组，然后new监听器。

```
builder.setMultiChoiceItems(Muilty_List, null, new DialogInterface.OnMultiChoiceClickListener()
        {
            @Override
            public void onClick(DialogInterface dialog, int which, boolean isChecked)
            {
                if(isChecked)
                {
                    Toast.makeText(MainActivity.this, "我喜欢上了"+Muilty_List[which], Toast.LENGTH_SHORT).show();
                }else
                {
                    Toast.makeText(MainActivity.this, "我不喜欢"+Muilty_List[which], Toast.LENGTH_SHORT).show();
                }
            }
        });
```
onclick方法中的which也是表示当前动作的下标，后面的`isChecked` 即是表示这个下标是被选中还是没被选中。
![这里写图片描述](http://img.blog.csdn.net/20160914152113509)

----------
## 列表对话框的用法##
同样的列表对话框也很简单，同样只需要动dialog的方法`setItems`

```
 builder.setItems(item_list, new DialogInterface.OnClickListener()
        {
            @Override
            public void onClick(DialogInterface dialog, int which)
            {
                Toast.makeText(MainActivity.this, "我不喜欢"+item_list[which], Toast.LENGTH_SHORT).show();
            }
        });
```
同样的数据源需要你提前定义好。效果![如下](http://img.blog.csdn.net/20160914153011084)

----------
## 自定义对话框 ##
凡是牵涉到自定义的多半都牵扯两个操作，一个是着做自定义的layout xml文件，另一个是用LayoutInflater加载布局为view，得到view后才能使用。
XML：`<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
    android:orientation="vertical">
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    >
    <EditText
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:hint="输入内容"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="提交"
        android:layout_marginLeft="1dip"/>
</LinearLayout>
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        android:layout_marginTop="10dip"/>
</LinearLayout>
`

----------

```
        LayoutInflater inflater=LayoutInflater.from(this);
        View view = inflater.inflate(R.layout.dialog_layout, null);
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setIcon(R.mipmap.ic_launcher);
        builder.setTitle("自定义对话框");
        builder.setView(view);
        AlertDialog dialog=builder.create();
        dialog.show();
```

----------
那么以上就是几种dialog的简单用法。

