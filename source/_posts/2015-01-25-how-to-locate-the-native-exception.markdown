---
layout: post
title: "定位Native异常"
date: 2015-01-26 21:08
comments: true
categories: android笔记
keywords: android,开发技巧,ndk,native
description: 本文主要介绍了2套定位native异常的工具。
---

##0x00 ndk-stack
Google从`NDK r6`开始提供这个方便的工具，帮助开发者定位Native 异常。和`ndk-build`一样，这个命令行工具被放在`NDK`安装目录下。作为Debug 工具，其操作方法也十分简单：只需在参数里指定包含符号表的`so`文件即可。

如果需要实时获取异常信息，我们可以直接运行：`adb shell logcat | ndk-stack -sym $PROJECT_DIR/obj/local/armeabi`；有的时候日志文件并不一定能直接看到，该工具也支持“异步”分析。这里我们首先运行：`adb shell logcat > android.log`来模拟“异步”获取日志。 在获得日志文件之后，我们拿它喂给命令行工具：`ndk-stack -sym $PROJECT_DIR/obj/local/armeabi -dump android.log`。

就这样，天书一般的日志文件瞬间变得友好可读了。

##0x01 addr2line以及objdump
对于有经验的程序员来说，可能更熟悉这2款工具。方法类似，结果相同，所以这里就不再赘述了。
