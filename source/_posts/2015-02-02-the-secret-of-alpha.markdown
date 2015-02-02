---
layout: post
title: "Alpha的秘密"
date: 2015-02-02 21:08
comments: true
categories: 性能优化, android笔记
keywords: alpha, 性能优化, 渲染
description: 本文介绍了一个关于alpha的优化渲染性能的小技巧。
---

Google 最近搞了个[Android Performance Patterns][1]， 挺好的，光这个名字，就已经给整个社区很好的定位了。其实**性能优化**这档子事吧，android 社区从来就有。只不过，从前都是只言片语，大伙也只能各自摸着石头过河。这一次Google不但在G+上搞了这个讨论区，还以“Patterns”的形式推出了[一套指导视频][2]，从渲染、内存、电池三大方面剖析了性能优化的方法、工具以及一些小的技巧。这一系列的动作，引发了各界不小的反应，我还看到有国内的开发者把这套视频翻译为"[android性能优化典范][3]"，着实给力呢。

既然有了这么好的氛围，这个博客也打算凑凑热闹，帮忙翻译、整理一些和性能优化有关的Tips。你可能不敢相信，一个简单的优化步骤能带来多大的性能提升？！让我们先看一张图：

{% img center /images/201502/alpha_contrast.png %}

如果你已经看过[Profile GPU Rendering][5] 的介绍，一定会被这幅对比图惊艳到。仅仅是一个`alpha` 属性的设置，两者的渲染性能差别如此之大。据[Roamn Nurik][6] 介绍`view`的alpha属性在合成之前需要放到屏幕外的缓冲区中作额外的渲染工作。但，如果只对`textcolor`作`alpha`属性的设置，**渲染性能立刻提升了10倍**。

与之类似，在`TextView`中使用`background set`,`Spannable text`以及让文字被选中，都会加重渲染的负担。了解渲染负担并合理的避免，应该才是性能优化的王道。

##参考资料

-   [improving performance by switching from view alpha to text color alpha][7]


  [1]: https://plus.google.com/u/1/communities/116342551728637785407
  [2]: https://www.youtube.com/playlist?list=PLOU2XLYxmsIKEOXh5TwZEv89aofHzNCiu
  [3]: http://hukai.me/android-performance-patterns/
  [5]: https://www.youtube.com/watch?v=VzYkVL1n4M8&list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE&index=4
  [6]: https://plus.google.com/113735310430199015092
  [7]: https://plus.google.com/u/1/+AndroidDevelopers/posts/3p3dHBszKvo
