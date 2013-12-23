---
layout: post
title: "android无线调试"
date: 2013-12-03 20:04
comments: true
categories: android笔记
keywords: wifi,debug,无线调试
description: 本文主要介绍了2种开启android 设备无线调试的方法。
---

使用无线调试对于我来讲，只有一个理由：爱惜手机电池。长时间的调试过程，使得我们的爱机必须通过USB 与电脑相连。长此以往，手机电池就会变得非常不经用。

很早之前，以为只有获得ROOT 权限的手机才能开启这个功能，直到有一天被我发现statckoverflow 上的一个[帖子][1]。（之所以会有这个错觉，是因为看到Google Play上提供的相关APP 都有root权限的声明）。那么，这里我就分别总结一下，不同情况下该如何使用**无线adb**

##如果手机拥有ROOT 权限
在shell 里执行以下命令：
``` bash 开启adb无线调试
su
setprop service.adb.tcp.port 5555
stop adbd
start adbd
```
这也是大部分完成无线调试APP的核心基础：在应用内使用Process 对象来执行这些命令。其中`start/stop adbd`，你可以通过手动开启/关闭调试模式来完成同样的目的。如果希望回到usb 模式，则应该将`tcp.port`修改回-1。

##如果手机没有ROOT 权限
-	事情并没有因为少一个权限而变得麻烦
-	首先将手机通过USB连接到PC
-	在命令行中执行`adb tcpip 5555`，即可开启无线adb了
-	想要恢复有线adb时，在保持手机与PC连接的前提下，命令行中执行`adb usb`即可

##连接与断开手机
-   开启无线adb后，想要通过WIFI连接手机，请先确保电脑和手机在同一网段
-   连接：adb connect android-device-ip
-   终止：adb disconnect


[1]: http://stackoverflow.com/questions/2604727/how-can-i-connect-to-android-with-adb-over-tcp
