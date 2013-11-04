---
layout: post
title: "Listview 子控件重复加载"
date: 2013-11-04 20:45
comments: true
categories: android笔记
---

这是一个常见的listview 资源文件的写法。
{% codeblock listview资源文件 lang:xml %}
    <ListView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/listView"
    />
{% endcodeblock %}

注意到`android:layout_height` 被设置为`wrap_content`，这似乎很合理。但如果我们留心的话可以发现，与这个listview 关联的adapter对象的`getView()`方法被**重复调用了好几遍**!这对于那些依赖listview 展示大量数据的应用来说，绝对是性能打击。

###解决方案：
将这个属性设置为`match_parent` 或者固定的数值

###原因
通过查看[AbsListView][1] 的代码，不难发现`mAdapter` 调用`getView()` 方法只有在无法从`RecycleBin`中获得子控件的时候才发生。从实验的log 来看，`getView()`方法被重复调用了3次。按道理来说通过前几次的调用，`RecycleBin`里应该已经有这个子控件了啊，为什么还会重复调用呢？

继续查看[Listview][2] 的代码，答案在`onMeasure()`方法里：由于`wrap_content` 并没有指定一个固定值，**系统需要通过尝试、计算来满足`wrap_content`**。每次尝试都会调用`onMeasure()`，进而调用父类的`obtainView()`，以至于最终调用`getView()`了


[1]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/java/android/widget/AbsListView.java
[2]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/java/android/widget/ListView.java
