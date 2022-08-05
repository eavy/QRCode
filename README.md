# ShareDemo
效果图：

![image](https://user-images.githubusercontent.com/66309387/183051478-8b547014-ba70-4c54-bdeb-c427fef3e6ad.png) 


![image](https://user-images.githubusercontent.com/66309387/183051764-8eee1931-50c1-41db-8aa0-24c501d1cb15.png)


将现有view生成bitmap，用于分享

# 原理
将布局绘制成位图并保存，然后调用系统自带的分享功能
# 添加存储权限

1. 在AndroidManifest.xml 声明权限
```java
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0273a849b73a499ab761c040ad292ad2.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzI0MDg2,size_16,color_FFFFFF,t_70)
2. 高版本Android 要需要动态申请权限
```java
    /**
     * 动态申请权限
     */
    private void checkNeedPermissions() {
        //6.0以上需要动态申请权限
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED) {
            //多个权限一起申请
            ActivityCompat.requestPermissions(this, new String[]{
                    Manifest.permission.CAMERA,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE,
                    Manifest.permission.READ_EXTERNAL_STORAGE
            }, 1);
        }
    }
```
# 截屏保存
```java
    /**
     * 截屏
     */
    public void captureScreen() {
        Runnable action = new Runnable() {
            @Override
            public void run() {
                /*获取windows中最顶层的view*/
                final View contentView = getWindow().getDecorView();
                try {
                    Log.e("chen", contentView.getHeight() + ":" + contentView.getWidth());
                    bitmap = Bitmap.createBitmap(contentView.getWidth(), contentView.getHeight(), Bitmap.Config.ARGB_4444);
                    contentView.draw(new Canvas(bitmap));
                    ByteArrayOutputStream byteOut = new ByteArrayOutputStream();
                    //将位图的压缩到指定的OutputStream，可以理解成将Bitmap保存到文件中！
                    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, byteOut);
                    File file = new File(Environment.getExternalStorageDirectory().getAbsolutePath() + "/test/");
                    if (!file.exists()) {
                        file.mkdirs();
                    }
                    savePic(bitmap, file.toString() + File.separator + "short.png");
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    try {
                        if (null != byteOut)
                            byteOut.close();
                        if (null != bitmap && !bitmap.isRecycled()) {
//                            bitmap.recycle();
                            bitmap = null;
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        try {
            action.run();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 保存位图到本地
     *
     * @param bitmap
     * @param strFileName
     */
    private void savePic(Bitmap bitmap, String strFileName) {
        Log.i("filepath:", strFileName);
        //Toast.makeText(MainActivity.this, strFileName, Toast.LENGTH_SHORT).show();
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(strFileName);
            if (null != fos) {
                boolean success = bitmap.compress(Bitmap.CompressFormat.PNG, 100, fos);
                fos.flush();
                fos.close();
                if (success) {
                    //Toast.makeText(MainActivity.this, "截屏成功", Toast.LENGTH_SHORT).show();
                    //分享
                    shareImage(strFileName);
                }
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
# 分享
```java

    /**
     * 将本地图片分享到第三方
     *
     * @param path 为本地文件绝对路径
     */
    public void shareImage(String path) {
        //解决Android7.0调用 Uri.fromFile() 报错FileUriExposedException
        StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
        StrictMode.setVmPolicy(builder.build());
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
            builder.detectFileUriExposure();
        }
        //由文件得到uri
        Uri imageUri = Uri.fromFile(new File(path));
        Log.d("share", "uri:" + imageUri);
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
        //图片
        shareIntent.setType("image/*");
        startActivity(Intent.createChooser(shareIntent, "分享到"));
    }
```
