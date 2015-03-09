---
layout: post
title: Android-Universal-Image-Loader详细配置
category: android
date: 2015-03-09
---
Android-Universal-Image-Loader是目前业内使用最广泛的异步图片加载框架之一，github项目地址是：[Android-Universal-Image-Loader](https://github.com/nostra13/Android-Universal-Image-Loader)。

导入jar包后，我们需要进行一些配置来帮助我们使用
#权限配置
在清单文件中加入以下权限

``` xml

<uses-permission android:name="android.permission.INTERNET" />  
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />  

```
#ImageLoaderConfiguration配置
*github文档上提供的帮助：*

ImageLoader **Configuration (`ImageLoaderConfiguration`) is global** for application. You should set it once.

All options in Configuration builder are optional. Use only those you really want to customize.<br />*See default values for config options in [Java docs for every option](https://github.com/nostra13/Android-Universal-Image-Loader/blob/master/library/src/com/nostra13/universalimageloader/core/ImageLoaderConfiguration.java#L199-599).*

``` java

// DON'T COPY THIS CODE TO YOUR PROJECT! This is just example of ALL options using.
// See the sample project how to use ImageLoader correctly.
File cacheDir = StorageUtils.getCacheDirectory(context);
ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(context)
		.memoryCacheExtraOptions(480, 800) // default = device screen dimensions
		.diskCacheExtraOptions(480, 800, null)
		.taskExecutor(...)
		.taskExecutorForCachedImages(...)
		.threadPoolSize(3) // default
		.threadPriority(Thread.NORM_PRIORITY - 2) // default
		.tasksProcessingOrder(QueueProcessingType.FIFO) // default
		.denyCacheImageMultipleSizesInMemory()
		.memoryCache(new LruMemoryCache(2 * 1024 * 1024))
		.memoryCacheSize(2 * 1024 * 1024)
		.memoryCacheSizePercentage(13) // default
		.diskCache(new UnlimitedDiscCache(cacheDir)) // default
		.diskCacheSize(50 * 1024 * 1024)
		.diskCacheFileCount(100)
		.diskCacheFileNameGenerator(new HashCodeFileNameGenerator()) // default
		.imageDownloader(new BaseImageDownloader(context)) // default
		.imageDecoder(new BaseImageDecoder()) // default
		.defaultDisplayImageOptions(DisplayImageOptions.createSimple()) // default
		.writeDebugLogs()
		.build();
		
```

可以看出，**Configuration (`ImageLoaderConfiguration`) is global**，我们可以在application中进行配置，并完成初始化

``` java

ImageLoader.getInstance().init(config);

```
*ImageLoaderConfiguration配置注释*

``` java

ImageLoaderConfiguration config = new ImageLoaderConfiguration  
    .Builder(context)  
    .memoryCacheExtraOptions(480, 800) // max width, max height，即保存的每个缓存文件的最大长宽  
    .diskCacheExtraOptions(480, 800, CompressFormat.JPEG, 75, null) // Can slow ImageLoader, use it carefully (Better don't use it)/设置缓存的详细信息，最好不要设置这个  
    .threadPoolSize(3)//线程池内加载的数量  
    .threadPriority(Thread.NORM_PRIORITY - 2)  
    .denyCacheImageMultipleSizesInMemory()  
    .memoryCache(new LruMemoryCache(2 * 1024 * 1024)) // You can pass your own memory cache implementation/你可以通过自己的内存缓存实现  
    .memoryCacheSize(2 * 1024 * 1024)    
    .diskCacheSize(50 * 1024 * 1024)    
    .diskCacheFileNameGenerator(new Md5FileNameGenerator())//将保存的时候的URI名称用MD5 加密  
    .diskCacheFileCount(100) //缓存的文件数量  
    .diskCache(new UnlimitedDiscCache(cacheDir))//自定义缓存路径  
    .defaultDisplayImageOptions(DisplayImageOptions.createSimple())  
    .imageDownloader(new BaseImageDownloader(context, 5 * 1000, 30 * 1000)) // connectTimeout (5 s), readTimeout (30 s)超时时间  
    .writeDebugLogs() // Remove for release app  
    .build();//开始构建  
	
```
##在ImageLoaderConfiguration的配置中
* memoryCache

``` java

.memoryCache(new LruMemoryCache(2 * 1024 * 1024))

```
是配置内存缓存的策略，默认的是*LruMemoryCache*(遵循Lru算法,强引用bitmap),还可以使用*UsingFreqLimitedMemoryCache*（如果缓存的图片总量超过限定值，先删除使用频率最小的bitmap，强引用与弱引用结合），*WeakMemoryCache*（弱引用缓存，bitmap的总大小没有限制，唯一不足的地方就是不稳定，缓存的图片容易被回收掉），也可以使用自己实现的缓存策略。

* diskCache

``` java

.diskCache(new UnlimitedDiskCache(cacheDir))

```
是用来定义外存缓存路径的，cacheDir是File，可以这样来配置：
``` java

File cacheDir = StorageUtils.getOwnCacheDirectory(getApplicationContext(), "MyAppName/Cache");  

```

* diskCacheFileNameGenerator

``` java

.diskCacheFileNameGenerator(new Md5FileNameGenerator())

```
是定义缓存文件的定义方式
可以调用的方法有 

``` java

new Md5FileNameGenerator() //使用MD5对UIL进行加密命名
new HashCodeFileNameGenerator()//使用HASHCODE对UIL进行加密命名

```
**已经进行disk缓存的网络图片，如果文件名（url）没有改变，不会再去请求下载**

#DisplayImageOptions 配置
*github文档上提供的帮助：*
**Display Options (`DisplayImageOptions`) are local** for every display task (`ImageLoader.displayImage(...)`).

Display Options can be applied to every display task (`ImageLoader.displayImage(...)` call).

**Note:** If Display Options wasn't passed to `ImageLoader.displayImage(...)`method then default Display Options from configuration (`ImageLoaderConfiguration.defaultDisplayImageOptions(...)`) will be used.

``` java

// DON'T COPY THIS CODE TO YOUR PROJECT! This is just example of ALL options using.
// See the sample project how to use ImageLoader correctly.
DisplayImageOptions options = new DisplayImageOptions.Builder()
		.showImageOnLoading(R.drawable.ic_stub) // resource or drawable
		.showImageForEmptyUri(R.drawable.ic_empty) // resource or drawable
		.showImageOnFail(R.drawable.ic_error) // resource or drawable
		.resetViewBeforeLoading(false)  // default
		.delayBeforeLoading(1000)
		.cacheInMemory(false) // default
		.cacheOnDisk(false) // default
		.preProcessor(...)
		.postProcessor(...)
		.extraForDownloader(...)
		.considerExifParams(false) // default
		.imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2) // default
		.bitmapConfig(Bitmap.Config.ARGB_8888) // default
		.decodingOptions(...)
		.displayer(new SimpleBitmapDisplayer()) // default
		.handler(new Handler()) // default
		.build();
		
```

*DisplayImageOptions配置注释*

``` java

DisplayImageOptions options = new DisplayImageOptions.Builder()  
		.showImageOnLoading(R.drawable.ic_launcher) //设置图片在下载期间显示的图片  
		.showImageForEmptyUri(R.drawable.ic_launcher)//设置图片Uri为空或是错误的时候显示的图片  
		.showImageOnFail(R.drawable.ic_launcher)  //设置图片加载/解码过程中错误时候显示的图片
		.cacheInMemory(true)//设置下载的图片是否缓存在内存中  
		.cacheOnDisk(true)//设置下载的图片是否缓存在SD卡中  
		.considerExifParams(true)  //是否考虑JPEG图像EXIF参数（旋转，翻转）
		.imageScaleType(ImageScaleType.EXACTLY_STRETCHED)//设置图片以如何的编码方式显示  
		.bitmapConfig(Bitmap.Config.RGB_565)//设置图片的解码类型//  
		.decodingOptions(android.graphics.BitmapFactory.OptionsdecodingOptions)//设置图片的解码配置 
		//.delayBeforeLoading(int delayInMillis)//int delayInMillis为你设置的下载前的延迟时间
		//设置图片加入缓存前，对bitmap进行设置  
		//.preProcessor(BitmapProcessor preProcessor)  
		.resetViewBeforeLoading(true)//设置图片在下载前是否重置，复位  
		.displayer(new RoundedBitmapDisplayer(20))//是否设置为圆角，弧度为多少  
		.displayer(new FadeInBitmapDisplayer(100))//是否图片加载好后渐入的动画时间  
		.build();//构建完成  
		
```
##在DisplayImageOptions配置中
* imageScaleType(ImageScaleType imageScaleType)
是设置图片的缩放方式
              EXACTLY :图像将完全按比例缩小的目标大小
              EXACTLY_STRETCHED:图片会缩放到目标大小完全
              IN_SAMPLE_INT:图像将被二次采样的整数倍
              IN_SAMPLE_POWER_OF_2:图片将降低2倍，直到下一减少步骤，使图像更小的目标大小
              NONE:图片不会调整
* displayer(BitmapDisplayer displayer)
是设置图片的显示方式
              RoundedBitmapDisplayer（int roundPixels）设置圆角图片
              FakeBitmapDisplayer（）这个类什么都没做
              FadeInBitmapDisplayer（int durationMillis）设置图片渐显的时间
　　　　　　　SimpleBitmapDisplayer()正常显示一张图片

#加载图片
* 普通带配置的加载

``` java

ImageLoader.getInstance().displayImage(imageUri, imageView，options); //imageUrl代表图片的URL地址，imageView代表承载图片的IMAGEVIEW控件，options代表DisplayImageOptions配置文件

```
* 带加载情况监听的加载

``` java

imageLoader.displayImage(imageUri, imageView, options, new ImageLoadingListener() {  
    @Override  
    public void onLoadingStarted() {  
       //开始加载的时候执行  
    }  
    @Override  
    public void onLoadingFailed(FailReason failReason) {        
       //加载失败的时候执行  
    }   
    @Override   
    public void onLoadingComplete(Bitmap loadedImage) {  
       //加载成功的时候执行  
    }   
    @Override   
    public void onLoadingCancelled() {  
       //加载取消的时候执行  
  
    }});  
	
```
* 带进度条的加载

``` java

imageLoader.displayImage(imageUri, imageView, options, new ImageLoadingListener() {  
    @Override  
    public void onLoadingStarted() {  
       //开始加载的时候执行  
    }  
    @Override  
    public void onLoadingFailed(FailReason failReason) {        
       //加载失败的时候执行  
    }      
    @Override      
    public void onLoadingComplete(Bitmap loadedImage) {  
       //加载成功的时候执行  
    }      
    @Override      
    public void onLoadingCancelled() {  
       //加载取消的时候执行  
    },new ImageLoadingProgressListener() {        
      @Override  
      public void onProgressUpdate(String imageUri, View view, int current,int total) {     
      //在这里更新 ProgressBar的进度信息  
      }  
    });  
	
```

* Acceptable URIs examples 可接受的URIs
"http://site.com/image.png" // from Web
"file:///mnt/sdcard/image.png" // from SD card
"file:///mnt/sdcard/video.mp4" // from SD card (video thumbnail)
"content://media/external/images/media/13" // from content provider
"content://media/external/video/media/13" // from content provider (video thumbnail)
"assets://image.png" // from assets
"drawable://" + R.drawable.img // from drawables (non-9patch images)

#如果出现OOM
* 减少配置之中线程池的大小，(.threadPoolSize).推荐1-5；
* 使用.bitmapConfig(Bitmap.config.RGB_565)代替ARGB_8888;
* 使用.imageScaleType(ImageScaleType.IN_SAMPLE_INT)或者.imageScaleType(ImageScaleType.EXACTLY)；
* 避免使用RoundedBitmapDisplayer.它会创建新的ARGB_8888格式的Bitmap对象；
* 使用.memoryCache(new WeakMemoryCache())或者不要使用.cacheInMemory();


#参考链接

[Android-Universal-Image-Loader,nostra13](https://github.com/nostra13/Android-Universal-Image-Loader)

[Android-Universal-Image-Loader 图片异步加载类库的使用（超详细配置）,vipra](http://blog.csdn.net/vipzjyno1/article/details/23206387)

[从源代码分析Android-Universal-Image-Loader的缓存处理机制,陈哈哈](http://www.cnblogs.com/kissazi2/p/3931400.html)
