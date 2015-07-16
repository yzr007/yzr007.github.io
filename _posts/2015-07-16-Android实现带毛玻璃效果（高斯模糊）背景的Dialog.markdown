---
layout: post
title: Android实现带毛玻璃效果（高斯模糊）背景的Dialog
category: android
date: 2015-07-16
---

最近换了工作，由于工作中要使用一些自己以前不是很了解的知识，就没有时间更新博客了。

由于最近做了一些很有意思的小demo，不吐不快，再加上还是认为技术需要沉淀和梳理，所以再次把写博客这件事拾起来。

已经不是第一次遇到设计师要求使用毛玻璃效果了，但是做带毛玻璃效果背景的Dialog还是第一次。考虑实现的逻辑，弹出Dialog之前对当前屏幕截图，做高斯模糊处理，设为Dialog背景，显示Dialog。


<!-- more -->

###屏幕截图核心代码

``` java
public static Bitmap takeScreenShot(Activity activity) { 
        // View是你需要截图的View 
        View view = activity.getWindow().getDecorView(); 
        view.setDrawingCacheEnabled(true); 
        view.buildDrawingCache(); 
        Bitmap b1 = view.getDrawingCache(); 
   
        // 获取状态栏高度 
        Rect frame = new Rect(); 
        activity.getWindow().getDecorView().getWindowVisibleDisplayFrame(frame); 
        int statusBarHeight = frame.top; 
   
        // 获取屏幕长和高 
        int width = activity.getWindowManager().getDefaultDisplay().getWidth(); 
        int height = activity.getWindowManager().getDefaultDisplay() 
                .getHeight(); 
        // 去掉标题栏 
        Bitmap b = Bitmap.createBitmap(b1, 0, statusBarHeight, width, height - statusBarHeight); 
        Matrix matrix = new Matrix(); 
        matrix.postScale(0.2f,0.2f); //长和宽放大缩小的比例 
        // 压缩 丢弃一部分像素点
        b = Bitmap.createBitmap(b,0,0,b.getWidth(),b.getHeight(),matrix,true);
        b = FastBlur.doBlur(b, 4, true); 
        view.destroyDrawingCache(); 
        return b; 
    } 
```

做压缩的目的是丢弃一部分像素点，保证做高斯模糊处理时的效率。既然要做高斯模糊，那么就意味着图片对精度的要求极低。

###FastBlur类(高斯模糊)

这个类来自于[开源项目StackBlur](https://github.com/kikoso/android-stackblur)的早期源码，感兴趣的可以使用最新的源码来替换。

``` java
public class FastBlur {
	
	    public static Bitmap doBlur(Bitmap sentBitmap, int radius, boolean canReuseInBitmap) {
	 
	        // Stack Blur v1.0 from
	        // http://www.quasimondo.com/StackBlurForCanvas/StackBlurDemo.html
	        //
	        // Java Author: Mario Klingemann <mario at="" quasimondo.com="">
	        // http://incubator.quasimondo.com
	        // created Feburary 29, 2004
	        // Android port : Yahel Bouaziz <yahel at="" kayenko.com="">
	        // http://www.kayenko.com
	        // ported april 5th, 2012
	 
	        // This is a compromise between Gaussian Blur and Box blur
	        // It creates much better looking blurs than Box Blur, but is
	        // 7x faster than my Gaussian Blur implementation.
	        //
	        // I called it Stack Blur because this describes best how this
	        // filter works internally: it creates a kind of moving stack
	        // of colors whilst scanning through the image. Thereby it
	        // just has to add one new block of color to the right side
	        // of the stack and remove the leftmost color. The remaining
	        // colors on the topmost layer of the stack are either added on
	        // or reduced by one, depending on if they are on the right or
	        // on the left side of the stack.
	        //
	        // If you are using this algorithm in your code please add
	        // the following line:
	        //
	        // Stack Blur Algorithm by Mario Klingemann <mario@quasimondo.com>
	 
	        Bitmap bitmap;
	        if (canReuseInBitmap) {
	            bitmap = sentBitmap;
	        } else {
	            bitmap = sentBitmap.copy(sentBitmap.getConfig(), true);
	        }
	 
	        if (radius < 1) {
	            return (null);
	        }
	 
	        int w = bitmap.getWidth();
	        int h = bitmap.getHeight();
	 
	        int[] pix = new int[w * h];
	        bitmap.getPixels(pix, 0, w, 0, 0, w, h);
	 
	        int wm = w - 1;
	        int hm = h - 1;
	        int wh = w * h;
	        int div = radius + radius + 1;
	 
	        int r[] = new int[wh];
	        int g[] = new int[wh];
	        int b[] = new int[wh];
	        int rsum, gsum, bsum, x, y, i, p, yp, yi, yw;
	        int vmin[] = new int[Math.max(w, h)];
	 
	        int divsum = (div + 1) >> 1;
	        divsum *= divsum;
	        int dv[] = new int[256 * divsum];
	        for (i = 0; i < 256 * divsum; i++) {
	            dv[i] = (i / divsum);
	        }
	 
	        yw = yi = 0;
	 
	        int[][] stack = new int[div][3];
	        int stackpointer;
	        int stackstart;
	        int[] sir;
	        int rbs;
	        int r1 = radius + 1;
	        int routsum, goutsum, boutsum;
	        int rinsum, ginsum, binsum;
	 
	        for (y = 0; y < h; y++) {
	            rinsum = ginsum = binsum = routsum = goutsum = boutsum = rsum = gsum = bsum = 0;
	            for (i = -radius; i <= radius; i++) {
	                p = pix[yi + Math.min(wm, Math.max(i, 0))];
	                sir = stack[i + radius];
	                sir[0] = (p & 0xff0000) >> 16;
	                sir[1] = (p & 0x00ff00) >> 8;
	                sir[2] = (p & 0x0000ff);
	                rbs = r1 - Math.abs(i);
	                rsum += sir[0] * rbs;
	                gsum += sir[1] * rbs;
	                bsum += sir[2] * rbs;
	                if (i > 0) {
	                    rinsum += sir[0];
	                    ginsum += sir[1];
	                    binsum += sir[2];
	                } else {
	                    routsum += sir[0];
	                    goutsum += sir[1];
	                    boutsum += sir[2];
	                }
	            }
	            stackpointer = radius;
	 
	            for (x = 0; x < w; x++) {
	 
	                r[yi] = dv[rsum];
	                g[yi] = dv[gsum];
	                b[yi] = dv[bsum];
	 
	                rsum -= routsum;
	                gsum -= goutsum;
	                bsum -= boutsum;
	 
	                stackstart = stackpointer - radius + div;
	                sir = stack[stackstart % div];
	 
	                routsum -= sir[0];
	                goutsum -= sir[1];
	                boutsum -= sir[2];
	 
	                if (y == 0) {
	                    vmin[x] = Math.min(x + radius + 1, wm);
	                }
	                p = pix[yw + vmin[x]];
	 
	                sir[0] = (p & 0xff0000) >> 16;
	                sir[1] = (p & 0x00ff00) >> 8;
	                sir[2] = (p & 0x0000ff);
	 
	                rinsum += sir[0];
	                ginsum += sir[1];
	                binsum += sir[2];
	 
	                rsum += rinsum;
	                gsum += ginsum;
	                bsum += binsum;
	 
	                stackpointer = (stackpointer + 1) % div;
	                sir = stack[(stackpointer) % div];
	 
	                routsum += sir[0];
	                goutsum += sir[1];
	                boutsum += sir[2];
	 
	                rinsum -= sir[0];
	                ginsum -= sir[1];
	                binsum -= sir[2];
	 
	                yi++;
	            }
	            yw += w;
	        }
	        for (x = 0; x < w; x++) {
	            rinsum = ginsum = binsum = routsum = goutsum = boutsum = rsum = gsum = bsum = 0;
	            yp = -radius * w;
	            for (i = -radius; i <= radius; i++) {
	                yi = Math.max(0, yp) + x;
	 
	                sir = stack[i + radius];
	 
	                sir[0] = r[yi];
	                sir[1] = g[yi];
	                sir[2] = b[yi];
	 
	                rbs = r1 - Math.abs(i);
	 
	                rsum += r[yi] * rbs;
	                gsum += g[yi] * rbs;
	                bsum += b[yi] * rbs;
	 
	                if (i > 0) {
	                    rinsum += sir[0];
	                    ginsum += sir[1];
	                    binsum += sir[2];
	                } else {
	                    routsum += sir[0];
	                    goutsum += sir[1];
	                    boutsum += sir[2];
	                }
	 
	                if (i < hm) {
	                    yp += w;
	                }
	            }
	            yi = x;
	            stackpointer = radius;
	            for (y = 0; y < h; y++) {
	                // Preserve alpha channel: ( 0xff000000 & pix[yi] )
	                pix[yi] = (0xff000000 & pix[yi]) | (dv[rsum] << 16) | (dv[gsum] << 8) | dv[bsum];
	 
	                rsum -= routsum;
	                gsum -= goutsum;
	                bsum -= boutsum;
	 
	                stackstart = stackpointer - radius + div;
	                sir = stack[stackstart % div];
	 
	                routsum -= sir[0];
	                goutsum -= sir[1];
	                boutsum -= sir[2];
	 
	                if (x == 0) {
	                    vmin[y] = Math.min(y + r1, hm) * w;
	                }
	                p = x + vmin[y];
	 
	                sir[0] = r[p];
	                sir[1] = g[p];
	                sir[2] = b[p];
	 
	                rinsum += sir[0];
	                ginsum += sir[1];
	                binsum += sir[2];
	 
	                rsum += rinsum;
	                gsum += ginsum;
	                bsum += binsum;
	 
	                stackpointer = (stackpointer + 1) % div;
	                sir = stack[stackpointer];
	 
	                routsum += sir[0];
	                goutsum += sir[1];
	                boutsum += sir[2];
	 
	                rinsum -= sir[0];
	                ginsum -= sir[1];
	                binsum -= sir[2];
	 
	                yi += w;
	            }
	        }
	 
	        bitmap.setPixels(pix, 0, w, 0, 0, w, h);
	 
	        return (bitmap);
	    }

}
```

Dialog的布局我就不赘述了，另外可以根据需要修改Style

``` xml
<item name="android:windowFullscreen">false</item> 
<item name="android:windowNoTitle">true</item>  
<item name="android:windowBackground">@android:color/transparent</item>
```

得到自己想要的效果。

最后，附上效果图
![普通背景](/res/img/blur1.png =320x240)![毛玻璃效果背景](/res/img/blur2.png =320x240)


本文出自[Yang](/)，转载时请注明出处及相应链接。



