---
layout: post
title: "蛋疼的BUG：binder事务异常"
date: 2014-11-11 15:15
comments: true
categories: android笔记
keywords: 解BUG, 程序崩溃, binder
description: 本文主要记录了寻找binder 事务异常的过程。
---

一句log"**!!! FAILED BINDER TRANSACTION !!!**"，伴随着App的崩溃，留下的是程序员深深的思考。定位到android 的源码，此log 出自`android_util_Binder.cpp`。

```c++

void signalExceptionForError(JNIEnv* env, jobject obj, status_t err, bool canThrowRemoteException)
{
    switch (err) {
        ...
        case FAILED_TRANSACTION:
            ALOGE("!!! FAILED BINDER TRANSACTION !!!");
            // TransactionTooLargeException is a checked exception, only throw from certain methods.
            // FIXME: Transaction too large is the most common reason for FAILED_TRANSACTION
            //        but it is not the only one.  The Binder driver can return BR_FAILED_REPLY
            //        for other reasons also, such as if the transaction is malformed or
            //        refers to an FD that has been closed.  We should change the driver
            //        to enable us to distinguish these cases in the future.
            jniThrowException(env, canThrowRemoteException? "android/os/TransactionTooLargeException": "java/lang/RuntimeException", NULL);
            break;
        ...
    }
}

```

很明显，按照注释的描述，我们遇到的问题就转换为了`TransactionTooLargeException`引起的程序崩溃了。去[官网](http://developer.android.com/reference/android/os/TransactionTooLargeException.html)一查，它给出的说明更清晰：远程调用的时候，参数和返回值都以Parcel 的形式保存在**Binder 事务的缓存对象**中。如果参数或返回值过大，则会抛出`TransactionTooLargeException`异常。


按照这个提示，我跟到驱动层的`binder.c`的[代码](https://android.googlesource.com/kernel/common.git/+/android-2.6.39/drivers/staging/android/binder.c)看了一下` binder_transaction`以及`binder_alloc_buf`方法，证实了前面注释代码所言非虚。另外，官方文档提到了一个1M 大小的buffer空间的限制，我翻了几个版本的代码来看，发现其实这个数值并没有写死。buffer 应该是不大于2M而不是它说的1M，但正如文档所说，由于binder 被所有跨进程通讯所共享，即使你的parcel 不大，但是使用binder 的人很多，也有可能造成上述异常。所以，这里解决问题的方法，应该是尽可能的传递小一点的parcel或者使用其他数据通讯方案。是的，说的就是你，不要用binder 传那么多那么大的高清图片了！


