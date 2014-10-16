---
layout: post
title: "android数据库升级tips"
date: 2014-10-16 22:18
comments: true
categories: android笔记
keywords: 技巧, android, 数据库, tips
description: 从android源码中发现的一段处理数据库升级代码。
---

###唯一不变的是变化
App 版本迭代的过程中，一定会遇到sqlite 数据库需要升级的情况。一个简单粗暴的解决方法，是不顾用户的数据，直接去重建各个数据表。显然，今天要分享的并不是这个粗暴的方法。

###常见的困惑
`SQLiteOpenHelper`提供了一个处理数据库升级的API`onUpgrade(final SQLiteDatabase db, int from, final int to) `。从语义上来看，升级策略的描述应该有一个起始点`from`，也还应该有一个终点`to`。但由于起始点的不确定性，致使这里的代码分支会略显复杂。老实讲，在写这篇日志之前，我都没有把握温柔细致的处理好这件事。

###灵感来源于android源码
`DownloadManager`也使用数据库存储下载状态。其子类`DatabaseHelper`将升级函数降维，把原本复杂的升级过程，分解为多个原子的升级过程，而这个原子过程是简单、确定的。直接上代码吧：

```java
@Override
 public void onUpgrade(final SQLiteDatabase db, int oldV, final int newV) {
    ...
    //循环体内，每次只升一个版本
    for (int version = oldV + 1; version <= newV; version++) {
                upgradeTo(db, version);
    }
}
        
```

```java

//Upgrade database from (version - 1) to version.
private void upgradeTo(SQLiteDatabase db, int version) {
    switch (version) {
        case 100:
            createDownloadsTable(db);
            break;
        case 101:
            createHeadersTable(db);
            break;
        case 102:
            addColumn(db, DB_TABLE, Downloads.Impl.COLUMN_IS_PUBLIC_API,
                              "INTEGER NOT NULL DEFAULT 0");
            break;
        case 105:
            fillNullValues(db);
            break;
        ...
        default:
            throw new IllegalStateException("Don't know how to upgrade to " + version);
        }
}
```

以上代码不仅逻辑清晰，而且还能很好的复用。比如在`onCreate`函数里需要初始化数，就可以直接调用`upgradeTo(db, CURRENT_VERSION)`.这样一来，无论是从哪个版本升级上来，都能平滑的处理版本问题了。


##参考资料
-  [DownloadProvider](https://android.googlesource.com/platform/packages/providers/DownloadProvider/+/master/src/com/android/providers/downloads/DownloadProvider.java#228)
