---
layout: post
title: 隐藏listview的headerview
category: android
date: 2015-03-05
---

应用新增了一个banner作为搞活动的入口，本来是放在listview的父布局上的，老板说要让banner和listview一起滚动，就将banner单独放在一个layout文件里，通过inflater动态加载并addHeaderView到listview里，但是在隐藏banner时出现了问题。
通常我们要隐藏View的时候会使用
```javascript
view.setVisibility(View.GONE)
//或者
view.setVisibility(View.INVISIBLE)
```
前者是隐藏的同时会重新布局，不再占用界面上的空间，后者只是单纯隐藏但是原来占用的空间会被空白填充。
这里，我给出banner的布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>

<FrameLayout  xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/banners"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:visibility="gone"
        >
        
    <ImageView 
        android:id="@+id/ad_banner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:scaleType="centerCrop"
        android:src="@drawable/default_banner"
        />
    <View 
        android:id="@+id/banner_exit"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:padding="5dp"
        android:layout_marginRight="8dp"
        android:layout_gravity="right|center_vertical"
        android:background="@drawable/banner_exit"
        />
</FrameLayout>

```
我见banner设为GONE，隐藏了，但是布局没有改变，原来banner占用的界面空间被空白填充了，效果与INVISIBLE完全一致，我惊了个呆。去网上寻找答案。
很多人给出的解决方案是：
```javascript
view.setVisibility(View.GONE);
view.setPadding(0, -view.getHeight(), 0, 0);
```
理论上是可行的，但是很low，而且非常可惜的是在我的手机上失效了。
所以我又找到了另一种切实可行的方案：
*答案就在这里*
--------------
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" >
    <FrameLayout
        android:id="@+id/banners"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:visibility="gone"
        >
        
    <ImageView 
        android:id="@+id/ad_banner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:scaleType="centerCrop"
        android:src="@drawable/default_banner"
        />
    <View 
        android:id="@+id/banner_exit"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:padding="5dp"
        android:layout_marginRight="8dp"
        android:layout_gravity="right|center_vertical"
        android:background="@drawable/banner_exit"
        
        />
    </FrameLayout>

</FrameLayout>
```
只要在外层嵌套一层空的父布局，直接设为GONE，完美隐藏，不会再有讨厌的空白占地方了。
原理的话，笔者说可能是setVisibility(View.GONE)对于 **根布局**是无效的，我还没有去求证，先放在这里，等我求证之后再补充。