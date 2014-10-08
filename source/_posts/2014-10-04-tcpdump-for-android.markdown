---
layout: post
title: "TCPDump for android"
date: 2014-10-04 23:41
comments: true
categories:  码农工具
keywords: tcpdump, android, 抓包, 工具, easydump
description: 一款开源的android抓包工具
---

###抓包要有效率

前段时间一直在做一款社交类答题的App。深有体会的是，不管是前期研究竞品，还是后期和服务端联调，抓包调试都是必不可缺少的环节，而且抓包的效率深深的影响着开发的效率。

由于REST API 的盛行，已经有大量现成的工具可以方便的进行抓包，比如大名鼎鼎的[Fiddler][1]， 以及免费且跨平台的[Rythem][2]。应用开发者只需要简单的[设置代理][3]，抓包信息即可尽收眼底。而非REST API的抓包则相对来说[就麻烦了许多][4]，每抓一次包既要碰电脑敲命令，又要弄手机进行测试操作，很是忧桑。因此，为了提升效率，本猿就想到基于[tcpdump][5] 重新定制一个工具。想要的效果也不复杂，一个全局的悬浮窗，给出抓包控制按钮即可。这样调试起来，就不用来回切换那么麻烦了。

###编译tcpdump
首先的工作当然是编译一个可用的tcpdump。虽然Android NDK 提供了方便的交叉编译工具，但是编译过程还是有[不少的坑][6]。比如android 不支持`setprotoent`以及`endprotoent`等。详细的编译过程，这里就不做讨论了。这里提供一个arm 版本的[tcpdump for android 下载][7]。（基于tcpdump-4.6.2.tar.gz，libpcap-1.6.2.tar.gz）

###开发工具
整个工具只有一个界面，一个全局窗口。代码在放在[Github 上][8]，当然你也可以直接从[Google play下载][9]、使用。噢，前两天看到一个[妹纸][10]发了一张图和抓包相关，而且还不错，这里也mark 一下。

{% img center /images/tcpdump_analyce.jpg %}


[1]: http://www.telerik.com/fiddler
[2]: https://github.com/AlloyTeam/Rythem
[3]: http://docs.telerik.com/fiddler/configure-fiddler/tasks/ConfigureForAndroid
[4]: http://www.kandroid.org/online-pdk/guide/tcpdump.html
[5]: http://www.tcpdump.org/
[6]: https://github.com/chatch/tcpdump-android/blob/master/build-tcpdump
[7]: http://pan.baidu.com/s/1vpMFG
[8]: https://github.com/shaomingbo/EasyDump
[9]: https://play.google.com/store/apps/details?id=de.mingbo.easydump
[10]: https://community.emc.com/people/Zhang%2CJiawen

