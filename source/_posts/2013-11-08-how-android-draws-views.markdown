---
layout: post
title: "视图的绘制"
date: 2013-11-08 14:48
comments: true
categories: android笔记
description: 关于视图绘制 的学习笔记
keywords: android, measure, layout, view, viewgroup
---

在[《Listview 子控件重复加载》](http://mingbo.de/blog/2013/11/04/listview-re-getview/)中谈到了`onMeasure()`方法，谈到了layout，但更细致的内容没有提到。这里找了一下相关资料来补充：

-   绘制过程是一个控件树遍历的过程，从根控件开始
-   ` ViewGroup`负责通知它的子视图，`View`负责自绘，按照顺序，父视图先于子视图绘制
-   绘制实际上是2个自顶向下的过程：measure 和layout。
-   经过measure 之后，每个`View`都会保存自己的尺寸，而在layout 过程中父视图会使用这些尺寸来摆放子视图
-   `View`对象的高宽会受到父视图的限制，以保证整个视图的正常显示
-   父视图会**多次调用子视图的measure方法**。比如：父视图会先计算不受约束的情况下，子视图的大小；如果子视图过大或者过小，父视图都会指定一个具体的值
-   Measure 过程还会涉及`ViewGroup.LayoutParams`以及`MeasureSpec`的设置
-   `ViewGroup.LayoutParams`被View 对象用来告诉其父控件，自己想如何被计算以及摆放，最基本的是指定长宽：
    1.  MATCH_PARENT, which means the View wants to be as big as its parent (minus padding)
    2.  WRAP_CONTENT, which means that the View wants to be just big enough to enclose its content (plus padding).
    3.  具体数值
-   MeasureSpec 指定一种计算模式：
    1.  UNSPECIFIED: This is used by a parent to determine the desired dimension of a child View.
    2.  EXACTLY: This is used by the parent to impose an exact size on the child. 
    3.  AT MOST: This is used by the parent to impose a maximum size on the child.


##参考资料
-   [How Android Draws Views](https://developer.android.com/guide/topics/ui/how-android-draws.html)
