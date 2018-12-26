---
title: onCreateContextMenu创建上下文菜单
date: 2016-09-17 13:11:57
tags: [Android]
---



所谓的上下文菜单就是一个你长按一个view，就会弹出一个菜单栏的东西，这玩意儿的使用非常简单。我们用listview来演示。

----------
## 创建上下文菜单 ##
由于Listview的所有选项名称都是放在一个List中的，所以我们需要自定义一个List如ArrayList来存放我们所有的选项名。

```
 private ArrayList<String>getDate()
    {
        ArrayList<String> arrayList = new ArrayList<>();
        for (int i=0;i<5;i++)
        {
            arrayList.add("File" + i);
        }
        return arrayList;
    }
```
在把listview设置完毕后，调用registerForContextMenu方法，即可将listview变为菜单。接着我们需要重写系统自带的`onCreateContextMenu` 这个方法一创建完毕就会得到menu对象，接下来你可以使用menu对象来设置标题，设置图标这些东西就不用说了吧。

----------
## 往上下文菜单里添加选项 ##
上下文菜单添加选项有两种办法：1.在代码中动态添加。2.在xml文件中静态注册。
在代码中添加就是简单调用add方法：

```
 public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo)
    {
        super.onCreateContextMenu(menu, v, menuInfo);
        menu.setHeaderTitle("File Oprete");
        menu.setHeaderIcon(R.mipmap.ic_launcher);
        menu.add(1,1,1,"复制");
        menu.add(1,2,1,"粘贴");
        menu.add(1,3,1,"剪切");
        menu.add(1,4,1,"重命名");
    }
```
add方法的第一个参数为groupid，第二个就是每个item的唯一标识符id，第三个参数就是优先顺序，第四个参数不用说了吧。
在xml文件中注册就需要在res/menu文件下添加一个xml文件：

```
 <item
        android:id="@+id/item1"
        android:title="复制"
        android:orderInCategory="100"
        ></item>

    <item
        android:id="@+id/item2"
        android:title="粘贴"
        android:orderInCategory="100"
        ></item>

    <item
        android:id="@+id/item3"
        android:title="剪切"
        android:orderInCategory="100"
        ></item>

    <item
        android:id="@+id/item4"
        android:title="重命名"
        android:orderInCategory="100"
        ></item>
```
然后接下来的事你就猜到了：

```
MenuInflater inflater = new MenuInflater(this);
        inflater.inflate(R.menu.contextmenu,menu);
```

----------
## 为上下文选项添加点击事件 ##
重写`onContextItemSelected` 方法就可以获得监听事件，很简单那。

```
 @Override
    public boolean onContextItemSelected(MenuItem item)
    {

        switch (item.getItemId())
        {
            case R.id.item1:

                Toast.makeText(MainActivity.this, "点击复制", Toast.LENGTH_SHORT).show();
                break;
            case R.id.item2:
                Toast.makeText(MainActivity.this, "点击粘贴", Toast.LENGTH_SHORT).show();
                break;
            case R.id.item3:
                Toast.makeText(MainActivity.this, "点击剪切", Toast.LENGTH_SHORT).show();
                break;
            case R.id.item4:
                Toast.makeText(MainActivity.this, "点击重命名", Toast.LENGTH_SHORT).show();
                break;
        }
        return super.onContextItemSelected(item);
    }
```
注意如果实在代码中添加的话，case 后就不应该是R.id.了，而应该是你add方法中的第二个参数。

效果如下：
![这里写图片描述](http://img.blog.csdn.net/20160917091834345)



