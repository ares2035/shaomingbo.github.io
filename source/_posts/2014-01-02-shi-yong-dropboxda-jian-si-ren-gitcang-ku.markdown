---
layout: post
title: "使用Dropbox搭建私人git仓库"
date: 2014-01-02 13:59
comments: true
categories: 码农工具
keywords: DIY,私人仓库,git,dropbox
description: 介绍使用dropbox等云存储方案搭建私人版本控制仓库。
---

###Why this?
-   有私人项目(不限于代码以及设计方案)
-   该项目（暂时）不公开
-   需要**版本控制**
-   虽然github 是一个不错的选择，但就项目目前状况而言premium 账号显得并不划算
-   项目备份以及研发环境的迁移
-   and so on


###How
-   就像stage area是working directory与本地仓库的缓存，本地仓库是working directory与中心仓库之间的缓存
-   我们把dropbox看成是中心仓库，那么working directory与中心仓库之间的缓存就是本地仓库了
-   建立本地仓库`git init <path to your project>`
-   构建stage area`git add .`
-   初始化提交`git commit -m 'repo init'`
-   创建dropbox 中心仓库`git init --bare ~/Dropbox/git/center.git`
-   建立远程连接`git remote add dropbox ~/Dropbox/git/center.git`
-   备份本地仓库`git push dropbox master`
-   拉取中心仓库`git pull dropbox master`
-   若开发环境发生了迁移，比如从公司换到了家里，`git clone ~/Dropbox/git/center.git`


###写在最后
-   好的云存储服务有很多，而dropbox是客户端最完备的一个。本文介绍的方法显然不限于dropbox一家。
-   `~/Dropbox/`是dropbox安装后的默认路径，不同的云存储的本地默认地址会不同
-   git子目录是我自己创建的，所以`center.git`也可以根据需求命名