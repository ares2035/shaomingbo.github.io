---
layout: post
title: "android内存泄露及分析工具"
date: 2014-10-24 14:42
comments: true
categories: android笔记
keywords: memory leak, android, 内存泄露, 分析, analyse
description: 本文介绍了android内存泄露的常见情况以及其分析工具
---

话说，任何开发环境，想要取得上乘的性能，内存优化是一个不可避免的话题。虽然android 开发过程中，有`GC`陪伴，但这并不意味着你可以完全忽略内存的分配与释放。这是因为，不恰当的代码仍然会导致内存泄露，以至于`GC`也无力回天。

###内存限制
为了维持多任务的运行环境，android 给每个运行的应用规定了一个内存上限（因不同设备的物理内存大小而变化，常见的有16M~48M不等）。每当你的应用企图在触碰上限的情况下分配内存，就会引发`OutOfMemoryError`。

###内存泄露
虚拟机中的内存泄露基本上就一条：代码残留了对象，让`GC`无法回收。也许这个情况一时半会不会翻起大浪，但也可能在下一秒，就触碰**内存限制**，导致程序崩溃。所以，内存泄露的BUG通常都非常隐蔽、随机、让人头大。

常见的泄露有以下几种：

-   `Cursor`没有关闭
-   `registerReceiver`之后没有成对的`unregisterReceiver`
-   I/O Stream没有关闭
-   `Context`泄露

前面3种情况，没啥好说的，在代码层面稍加注意即可避免。最后一种情况可能有些变种需要注意，这里使用2个例子尝试着描述一下。首先借用Romain Guy的例子：


```java
private static Drawable sBackground;

@Override
protected void onCreate(Bundle state) {
  super.onCreate(state);
  
  TextView label = new TextView(this);
  label.setText("Leaks are bad");
  
  if (sBackground == null) {
    sBackground = getDrawable(R.drawable.large_bitmap);
  }
  label.setBackgroundDrawable(sBackground);
  
  setContentView(label);
}
```

万恶的静态变量出现了，引用关系`Drawable`-> `TextView`-> `Activity`导致整个Activity 无法被回收。此外，经常被讨论的Webview 泄露，情况也是类似的。解决方法可以参考这个[帖子][4]。这里要谈的另外一种`Context`泄露是由于非静态内部类引用导致的:

```java

public class DemoActivity extends Activity {
    class LeakRUN implements Runnable {

        @Override
        public void run() {
            try {
                TimeUnit.DAYS.sleep(365);
            } catch (InterruptedException e) {
            }
        }
    }

}
```

正确的处理这种泄露应当仿造`ViewRoot`中的内部类`W`那样使用[弱引用][2]。这么做的好处是，一来不会出现空指针，二来不会导致内存泄露。

##内存工具
尽管上面罗列了种种常见的泄露情况，但实际开发中遇到的现象总是千奇百怪的，解决泄露的方法也会因情况而异。所以，这里更重要的是需要了解如何使用工具来发现应用内存是如何泄露的。

###Logcat
越简单的方法，往往越直接有效。Logcat 提供实时的虚拟机日志，每当垃圾回收发生的时候，我们会看到这样的日志`D/dalvikvm: <GC_Reason> <Amount_freed>, <Heap_stats>, <External_memory_stats>, <Pause_time>`。其中，垃圾回收的原因、回收内存的大小这些并不是现在关注的重点，我们的兴趣点应该放在**堆的状态**上。这个状态通常会提供两个数值，“空闲内存的比例”以及“内存占用的比例”。如果后者在每次垃圾回收之后，持续上升，**那么几乎就可以断定出现内存泄露了**。

###Monitor
**Monitor**是一个工具大集合，开发者可以从`Android studio/ Eclipse`直接找到它的入口，调试人员也可以直接从目录`<sdk>/tools/`下找到它。比起Logcat 被动的查看log, `Heap Update`提供了可视化的内存数据界面以及垃圾回收的操作接口。连续的在观察数据、回收内存以及应用交互的过程中往返，可以帮助我们定位**“哪些操作”引起了内存激增**。`Allocation Tracker`则提供了更为精确的**代码级定位**。

{% img center /images/201410/monitor-vmheap.png %}

###adb
adb 命令行工具提供的内存查询接口则更为灵活，信息也更加丰富。一行命令`adb shell dumpsys meminfo <package_name>`即可完成操作。有精力的开发者完全可以依此定制一款自己的工具来帮助内存泄露的定位。一般来说，我们可以关注`Pss Total`以及`Private Dirty`这两项数据来掌握内存信息，而`Objects`栏目的数据可以帮助引导我们更直观的发现泄漏的情况。

{% img center /images/201410/dumpsys_meminfo.png %}

###Heap Dump
最有效的工具，肯定需要最后出场。在Monitor 中选择`Dump HPROF file`，可以导出整个设备的虚拟机内存信息。由于该文件并不是传统的java 虚拟机内存文件，因此需要使用android sdk 中的`hprof-conv`工具对其进行转换。对于转换后的文件，可以使用[Eclipse Memory Analyzer Tool (MAT)][3]进行分析。该工具提供两种不同的视图（`Histogram`和`Dominator tree`）分别从不同的视角帮助你分析内存。值得一提的是，如果需要更为精确的导出时机的话，还可以在项目的代码中进行控制。


以上工具的具体使用方法，可以参考以下链接的文献，这里就不再赘述了。

##参考资料
-   [Android Memory Management: Understanding App PSS](http://www.littleeye.co/blog/2013/06/11/android-memory-management-understanding-app-pss/)
-   [Memory management for Android Apps](http://www.youtube.com/watch?v=_CruQY55HOk)
-   [Avoiding memory leaks](http://android-developers.blogspot.com/2009/01/avoiding-memory-leaks.html)
-   [Managing your app's memory](http://developer.android.com/training/articles/memory.html)
-   [Investigating Your RAM Usage](https://developer.android.com/tools/debugging/debugging-memory.html)


[1]: http://developer.android.com/guide/components/processes-and-threads.html
[2]: http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/2.1_r2/android/view/ViewRoot.java#ViewRoot.W
[3]: http://www.eclipse.org/mat/downloads.php
[4]: http://stackoverflow.com/questions/3130654/memory-leak-in-webview
