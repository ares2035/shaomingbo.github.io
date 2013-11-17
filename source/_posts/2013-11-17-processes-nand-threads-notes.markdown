---
layout: post
title: "进程与线程"
date: 2013-11-17 15:57
comments: true
categories: android笔记
description: 关于进程与线程的学习笔记
keywords: android, process, thread, android:multiprocess, 优先级
---

Android 组件具有天生的跨进程特性，因此，android 应用开发者通常是不需要关注进程概念的。但这也往往导致我们忽视了进程的一些细节问题。这里，就目前的认识做一些小结。

Android 是以**进程对用户的重要性**为依据来管理进程的。其重要性分为5个层级，重要性越低的进程越容易被系统干掉。

-   Foreground process
-   Visible process
-   Service process
-   Background process
-   Empty process

###Foreground process
优先级最高，即使在系统内存吃紧的情况下不到万不得已，是不会杀这类进程的。如果这类进程被杀，通常是因为用户界面卡得不能动弹而被迫执行的。这类进程所包含的组件通常满足以下特征：

    1.  activity 正在与用户进行交互
    2.  service 绑定的activity 正在与用户交互
    3.  service 执行了`startForeground()`
    4.  service 正在执行生命周期回调函数
    5.  BroadcastReceiver 正在执行`onReceived()`

###Visible process
优先级次之，系统在保证Foreground process 的前提下，保证这类进程的运行。这类进程虽不直接与用户交互，但用户能在界面上看到与之相关的组件。这些组件通常会有以下特征：

    1.  activity 在Paused 状态，处于可视但不可交互的状态
    2.  service 绑定的activity 处于paused 状态

###Service process
虽然没有与用户直接交互，但它们所做做的事情与用户息息相关。所以系统会按照优先级，在保留上述进程的前提下，尽可能的保留此类进程。不符合上面描述的Service 进程都属于Service Process。

###Background process
随时会被系统回收。为了提升用户体验，系统使用LRU 缓存维护这类进程，使得用户最近使用的进程推迟回收。这类进程通常只持有不可视组件，如paused 状态下的activity。

###Empty process
这类进程的存在仅仅只是为了让启动进程看起来更快一些，也因此更容易被系统回收。

###android:multiprocess 属性
默认情况下，该设置为false，表示该组件会在定义组件的应用进程中运行。当第三方应用调用该组件时，会有2个进程启动：第三方应用以及定义组件的应用。

如果该属性设置为true，表示允许该组件“嫁”出去：当第三方应用调用该组件时，会直接在该进程中构造组件。这么做会让组件在整个系统中的数量增加，但与调用者之间的交互更为紧密。值得一提的是，在实际开发中，这么做需要考虑并发的复杂性。

###零散的知识点

*   一个进程如果包含多个组件，那么，该进程取组件中优先级最高的作为自己的优先级
*   一个进程的优先级会因为其他进程的依赖而提升
*   由于service 比后台进程优先级高，一些长时间操作在service 中运行更有安全感
*   IBinder、 ContentProvider 的方法通常要注意线程安全

###参考资料
-   [Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html)
-   [activity属性](https://developer.android.com/guide/topics/manifest/activity-element.html#multi)



