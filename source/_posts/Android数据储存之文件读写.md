---
title: Android数据储存之文件读写
date: 2016-09-02 13:02:21
tags: [Android,数据库]
---



Android的文件读写主要是通过操作输入输出流来完成的，例如这个例子，我要在EditText中输入一段字符，并在Textview显示出来。

```
<EditText
        android:id="@+id/editText1"
        android:layout_width="wrap_content"
        android:layout_height="200dp"
        android:layout_alignParentLeft="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"
        android:ems="10"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="写入并读取出来"
        android:id="@+id/writebutton"
        android:layout_below="@+id/editText1"
        android:layout_centerHorizontal="true" />
```
关键就是我们要在按钮点击时调用WriteFile()和ReadFile()方法。

----------
## WriteFile()写文件 ##

```
 public void WriteFile(String content) {
        try {
            FileOutputStream  fileOutputStream = openFileOutput("a.txt", MODE_PRIVATE);
            fileOutputStream.write(content.getBytes());
            fileOutputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
Android自身时给了一个函数来打开输出流的——openFileOutput()，如果只以名字加格式作为param1的话，姐会自动创建在默认目录下，你也就可以指定路径，param2就是打开方式。接下来就是调用write函数，将字符串以byte的形式写入，最后记得所有流都需要手动关闭。

----------
## ReadFile()读取文件 ##`public String ReadFile() {
        String content = null;
        try {
            FileInputStream fileInputStream = openFileInput("a.txt");
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];//每次都读1k
            int len = 0;
            while ((len = fileInputStream.read(buffer)) != -1) {
                outputStream.write(buffer, 0, len); //写一个buffer这么大，并且从0开始写
            }
            content = outputStream.toString();
            fileInputStream.close();
            outputStream.close();
    
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return content;
    }`
同样的Android也给输入流提供了函数，这个函数很简单，参数也只有你想要读取的文件名，ByteArrayOutputStream的文档是这么解释这个类的`This class implements an output stream in which the data is written into a byte array.` 也就是说它更像是一个byte类型的数组。定义buffer数组的原因是控制每一次的输入大小，1k。接下来调用ByteArrayOutputStream的write的方法将读取到buffer数组的值写入ByteArrayOutputStream中，然后转换为字符串返回就行了。

----------
## 点击按钮事件读写 ##
接下来的事情就很简单了，给按钮增加点击事件，

```
 @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.writebutton:
                String content = editText.getText().toString();
                WriteFile(content);
                textView.setText(ReadFile());
                break;
            default:
                break;
        }
    }
```
那么呈现在界面上就是这样的也就大功告成了。
![这里写图片描述](http://img.blog.csdn.net/20160902093315310)

