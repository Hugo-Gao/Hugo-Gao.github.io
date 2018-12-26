---
title: Android调用相机和相册获取图片并存入数据库
date: 2016-10-27 13:18:28
tags: [Android]
---



最近在做项目的时候有一个需求，是要从相机中或相册中获取图片，而且还要将其存入SQLite，最开始的时候我想的是直接将图片存入数据库，但是后来在Google上发现不行，sqlite不支持这种类型，但是我看到了它支持Blob这种类型，也就是二进制，这种类型可以储存图片和视频，既然最基本的储存解决了，那么就开始动手写代码了。

----------
## 直接用模板代码调用相机和相册 ##
调用相机和相册是有模板代码的，可以考虑以后把它写成一个自己的库，方便以后的项目中调用。其大致思路就是：
1.  确定好图片名称，由于图片名称肯定不能重复，所以这里直接用当前时间加上后缀来命名。
2.  获取File路径，创建File对象，也就是你要将图片放在哪。
3.  将File对象转换为Uri对象。
4.  利用Intent放入要执行的动作，这里从相册选取和拍照选取图片有所不同。
5.  startActivityForResult直接启动Intent，在onActivityResult 接受数据。
6.  通过requestCode判断是拍照还是打开相册。
7.  如果是拍照就准备Intent启动裁剪工作，依然是使用startActivityForResult来启动裁剪程序获得图片，从相册选取得到照片也是一样的。
8.  最后将图片转换为Byte[]类型，存入数据库。
9.  刷新UI ，展示获得的图片。

```
//此为启动相机
takePhotoButton.setOnClickListener(new View.OnClickListener()
                {
                    @Override
                    public void onClick(View v)
                    {
                        String photoName = bean.getDateInfo() + "_image.jpg";
                        Log.d("haha", "photoName is " + photoName);
                        outputImage = new File(Environment.getExternalStorageDirectory()+"/ASimpleCount/", photoName);
                        try
                        {
                            if(outputImage.exists())
                            {
                                outputImage.delete();
                            }
                            outputImage.createNewFile();
                            Log.d("haha", "创建图片存储目录");
                        } catch (IOException e)
                        {
                            Log.d("haha", "创建目录抛出异常");
                            e.printStackTrace();
                        }
                        imageUri = Uri.fromFile(outputImage);//将文件路径转化为Uri对象
                        Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
                        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                        intent.putExtra("name", bean.getName());
                        startActivityForResult(intent,TAKE_PHOTO);
                        materialDialog.dismiss();
                    }
                });
```

```
//此为启动相册
chooseGalleryButton.setOnClickListener(new View.OnClickListener()//选择从相册选择相片
                {
                    @Override
                    public void onClick(View v)
                    {
                        String photoName = bean.getDateInfo() + "_image.jpg";
                        outputImage = new File(Environment.getExternalStorageDirectory()+ "/ASimpleCount/", photoName);
                        try
                        {
                            if(outputImage.exists())
                            {
                                outputImage.delete();
                            }
                            outputImage.createNewFile();
                        } catch (IOException e)
                        {
                            e.printStackTrace();
                        }
                        imageUri = Uri.fromFile(outputImage);//将文件路径转化为Uri对象
                        Intent intent = new Intent(Intent.ACTION_PICK);
                        intent.setType("image/*");
                        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                        intent.putExtra("name", bean.getName());
                        startActivityForResult(intent, CHOOSE_PHOTO);
                        materialDialog.dismiss();
                    }
                });

            }
        });
```

```
 @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data)
    {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode){
            case TAKE_PHOTO:
                    if(resultCode==RESULT_OK)
                    {
                        Intent intent = new Intent("com.android.camera.action.CROP");
                        intent.setDataAndType(imageUri, "image/*");
                        intent.putExtra("scale", true);
                        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                        startActivityForResult(intent,CROP_PHOTO);
                    }
                break;
            case CROP_PHOTO:
                    if (resultCode == RESULT_OK)
                    {
                            bitmap = Compressor.getDefault(this).compressToBitmap(outputImage);
                            bean.setPicInfo(BitmapHandler.convertBitmapToByte(bitmap));
                            /**
                             * 数据获取完毕，增加卡片
                             */
                            addNewCard();
                    }
                break;
            case CHOOSE_PHOTO:
                ContentResolver resolver = getContentResolver();
                Uri imgUri = data.getData();
//                try
//                {
//                    bitmap = MediaStore.Images.Media.getBitmap(resolver,
//                            imgUri);
                    bitmap= tool.ImageUtil.getScaledBitmap(this, imgUri, 612.0f,  816.0f);
                    bean.setPicInfo(BitmapHandler.convertBitmapToByte(bitmap));
//                } catch (IOException e)
//                {
//                    e.printStackTrace();
//                }
                addNewCard();
                break;
        }
    }
```

```
 private void addNewCard()
    {
        saveBeanToDataBase();
        Log.d("haha", "在addcard方法中"+bean.toString());
        showRecyclerView();
    }
```

----------
## 问题的发生及解决 ##
当然不会这么一帆风顺，在调试app时发现从相册取出图片可以正常工作，但是调用相机拍照时，裁剪完毕点击确定时，app进入黑屏状态，停留10多秒后直接闪退，待我马上再次点开app时系统报错。

```
01-29 13:41:56.520: W/CursorWindow(4121): Window is full: requested allocation 5140987 bytes, free space 2096617 bytes, window size 2097152 bytes
01-29 13:41:56.520: E/CursorWindow(4121): Failed to read row 0, column 0 from a CursorWindow which has 0 rows, 9 columns.

01-29 13:43:30.932: W/System.err(4121): java.lang.IllegalStateException: Couldn't read row 0, col 0 from CursorWindow.  Make sure the Cursor is initialized correctly before accessing data from it. 
01-29 13:43:30.932: W/System.err(4121):     at android.database.CursorWindow.nativeGetLong(Native Method) 
```
这是什么？从来没有见过这种报错，不过CursorWindow倒是给我提醒可能是数据库的问题，我不信邪，就卸了app重新运行，这次发现从相册选取照片也是有事行有时不行，我还没有遇到过这么怪的事，于是我就把上段报错信息google了一下，马上就在StackoverFlow上找到了答案，里面有一个和我一模一样的问题，下面已有人解决了：
> http://stackoverflow.com/questions/21432556/android-java-lang-illegalstateexception-couldnt-read-row-0-col-0-from-cursorw


![这里写图片描述](http://img.blog.csdn.net/20161027193000230)

原来是数据库的读取中出现了问题，Cursor对象只能够存储1MB的数据（网上有人说是1MB，我验证了一下，好像确实是1MB），多了会发生这样的报错。为了验证这个原因，我想之前从相册选取照片有时成功有时失败的问题，我猜想可能是选取了大于1MB的图片和小于1MB的图片的问题，为了验证这个猜想我准备了两张不同大小的照片，果然在选取小的那张时就成功了，大的系统就崩溃了。
     现在问题已经找出来了，那么在我面前有两条路。
1.  重新设计数据库结构，改直接储存图片为间接储存图片的Uri地址。
2.  不用改变数据库结构，将图片压缩至1MB内，存入数据库。
       两种方法各有好处，第一种方法的好处是数据库本就不适合储存数据大的文件，间接储存可以提高数据库读写效率。但是坏处是数据的整体性不强，如果用户在手机里把图片删了，那么从数据库里读取就会报错，而且对于我现在来说代码改动太大了，不利于维护；第二种方法的好处是整体性强，直接存入数据库，就算用户在手机里删除了图片，也不怕，数据库里还有。但是相应的效率就会下降。
      我当即决定采用第二种方法，我实在是不想动代码太多了，但是我又不懂图片压缩算法，不可能去现学吧。于是我马上想到了Github这个牛人云集的地方，我在github搜索“Android Compress”于是马上就有写好的库给我调用了。在这里再次感谢

> https://github.com/zetbaitsu/Compressor
> 提供的图片压缩算法。
> 好了仅仅只需要一行代码就可以压缩图片了`bitmap= tool.ImageUtil.getScaledBitmap(this, imgUri, 612.0f,  816.0f);`  但是最开始这个库并没有给我提供Uri转Bitmap的方法，而是只有File转Bitmap，我不信这个邪，于是我点进源代码看。

```
 public Bitmap compressToBitmap(File file) {
        return ImageUtil.getScaledBitmap(context, Uri.fromFile(file), maxWidth, maxHeight);
    }
```
原来是调用了ImageUtil.getScaledBitmap方法，而这个方法是调用的uri，之不过多了一个将File转化为Uri的过程，但是我现在不需要这个过程怎么办呢？于是我机智的修改了下源代码为我所用，那么一切都OK了。
      最后运行程序，不管是相册还是相机都运行得很好了。


----------
## 反思总结 ##
今天的这个找bug我收益良多，总结起来就是，注意报错信息，利用好Google，Stackoverflow，并注意多调试验证从网上看到的答案。




