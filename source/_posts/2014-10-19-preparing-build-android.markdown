---
layout: post
title: "警惕libgl1-mesa-glx:i386"
date: 2014-10-19 15:13
comments: true
categories: android笔记
keywords: ubuntu, android, apt-get, install, lib
description: 描述在ubuntu 12.04.5环境下，准备android的编译环境时所遇到的坑
---

编译android工程应该来说没啥技术含量，按照官方的guide line一步一步的做，基本上都还OK。但，之所以想写个日志，主要是因为被这个工程的编译环境搞得有点抓狂了。起初是在`osx`上折腾，之后发现各种依赖关系搞起来太费神，就开始了转投传说中最稳定、顺畅的ubuntu 12.04了。万万没有想到，在虚拟机上跑了4个小时，最后给我看这个：

{% img center /images/201410/built_android_failed.png %}

刚查到[解决方案][1]的时候，打算给虚拟机更大的内存跑编译任务，于是重启了。重启之后，蛋疼的卡在了logo界面。一顿排查，发现原来是在准备编译环境的时候出了问题。

```bash
sudo apt-get install git gnupg flex bison gperf build-essential \
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
  libgl1-mesa-dev g++-multilib mingw32 tofrodos \
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386
```

###症结所在
Ubuntu 12.04.5 上执行`sudo apt-get install libgl1-mesa-glx:i386 `时，会提示你安装`libgl1-mesa-dri:i386`。如果按照这个建议执行，你会发现安装程序会帮你删除一批不兼容的软件，其中包括`xorg`，最终导致你无法进入桌面。

{% img center /images/201410/remove_xorg_when_install_lib.png %}

###解决办法
将`libgl1-mesa-glx:i386`替换为`libgl1-mesa-glx-lts-<release>:i386`，其中release 根据ubuntu的版本可以是`Quantal`、`Raring`、`Saucy`以及`Trusty`。我自己安装的是`libgl1-mesa-glx-lts-Trusty:i386`。至此，该问题完美解决。

##参考资料
- [LTS Enablement Stacks](https://wiki.ubuntu.com/Kernel/LTSEnablementStack)

[1]: http://blog.csdn.net/g_r_u_b/article/details/8644745
