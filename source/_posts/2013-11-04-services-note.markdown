---
layout: post
title: "Service 心经"
date: 2013-11-04 15:21
comments: true
categories: android笔记
---
Service 作为一个“看不见”的android 组件，天然的就应该为Android 程序去处理那些费事费力的苦活。但即使这样，作为四大组件之一的它依然运行在主线程上。也就是说，那些理所当然的“负担”如果**直接运行在Service 上必然会影响程序的UI 性能**。因此，我们通常需要与之对应的后台线程来完成工作。

与其他组件一样，要想Service 正常运行，就必须在`AndroidManifest.xml` 中对其进行声明。如果Service 作为一个开放组件（允许外部程序调用），则还需要设置Intent Filter 来支持隐式调用；反之，如果它只在程序内运行，则可省去Filter 设置，更妥当的做法是将`export`属性设置为false。

总的来说，Service 有两种形式：**命令启动类(Started)** 与 **绑定类(Bound)**。两种形式之间并不总是非此既彼的关系，同一个应用的同一个Service 可以同时满足这两种形式，而这一点完全取决于应用逻辑。

###命令启动类（started）的Service
-   一旦被启动，如果没有其他组件调用`stopService()`而且自己也没有调用`stopSelf(int)`，那么它将会一直运行下去（除非系统内存吃紧）
-   `onStartCommand()`有三类返回值，分别代表Service 在执行完该函数之后，如遭遇系统Kill 的不同策略：
    1.  `START_NOT_STICKY`，除非有新的命令，不然不会重新构建Service
    2.  `START_STICKY`，不死Service。如果系统资源允许，会重新构建Service并调用`startCommand()`，但并不会重新传递最后一次发过来的intent。（适用于不care 命令，而care 启没启动的操作，比如恢复音乐播放）
    3.  `START_REDELIVER_INTENT`, 对intent 负责到底。在系统资源允许的情况下，重新构建service并将最后一次发送的intent 通过参数传递给`startCommand()`。（适用于care 命令，比如恢复下载某个文件）
-   和绑定类服务不同，如果希望得到交互结果，则必须使用Broardcast传递
-   `startCommand()` 会有并发的情况，但`stopService()` 只会执行一次。所以在具体应用场景中，我们需要考虑命令Service们能不能、该不该被中途停止。如果得到的答案是否定的，那么我们应该尝试使用`stopSelf(int)`来管理service，其参数为希望停止的service ID。

###简化的命令服务IntentServie
如果你不希望Service 处理并发请求，那么IntentService 是不二的选择。之所以这么说，是你几乎只需要三步就可以实现一个为你完成后台操作的单线程服务：1，继承IntetnService；2，覆盖`onHandleIntent()`；3，在构造函数中初始化父类。那么IntentService 会帮你做到：
-   创建一个后台线程
-   创建一个消息队列来执行任务
-   当没有任务的时候自动停止

如果希望自己覆盖某些Service 的方法，**那么一定要记得调用super**，否则会导致一些意想不到的结果出现。

###绑定类(Bound)服务
-   `IBinder` 表示客户端与服务端交互的接口
-   三种提供Binder 接口的方法：
    1.  直接继承Binder 类，提供API 给客户端调用。这些API 既可以由Binder 子类来实现，也可以通过API 返回该服务或其他服务的实例，由服务本身提供的方法去完成调用；**这种方法适用与私有服务**。
    2.  使用Messenger返回binder。该方法的本质是AIDL，提供了一种线程安全的跨进程通讯方案。**这种方法适用于服务与调用组件不在同一个进程的情况**。详见`参考代码`
    3.  如果服务不但跨进程，而且还希望处理并发请求，则应该使用`AIDL`来实现
-   多个客户端可以绑定同一个服务，但`onBind()` 只会在第一个客户端绑定的时候被调用。之后绑定的客户端虽然会获得同样的binder，但已经不会再调用`onBind()`了
-   四大组件中除了`Broardcast Receiver`之外都可以绑定服务
-   `onServiceDisconnected()`只会在服务崩溃的时候调用，**而并不会在unbind 的时候被调用**
-   bind 应与unbind 成对出现，服务与其绑定的客户端组件共存亡。如果交互工作完全结束的话，可以尽早的结束绑定，而不一定要等到组件消亡(比如某浪sso 接口那样)
-   bind 与unbind 不应该出现在`onResume()` 以及`onPause()`里，这样会使得服务的绑定与解绑操作过于频繁
-   绑定类服务又恰巧被调用了`startService()`，则只有同时满足以下条件，服务才会终止：1，没有客户端组件绑定该服务；2，服务调用了`stopService()` 或者`stopSelf(int)`
-   如下图所示，只有当`unBind()`返回true 的时候，下次绑定的时候，系统才会调用`onRebind()`，否则还会继续调用`onBind()`
-   所有对象均为跨进程引用计数
-   如果服务连接丢失的话，容易引发`DeadObjectException`

![服务的生命周期](https://developer.android.com/images/fundamentals/service_binding_tree_lifecycle.png)

###前台服务
-   前台服务能够极大的提升不被杀掉的概率
-   前台服务必须有与之对应**ON-GONING**通知显示
-   `stopForeground()`并不会停止服务；而停止服务后，通知自然消亡

###AIDL
-   Stub 继承自Binder 还实现了aidl 中的接口，除此之外还提供类似`asInterface(IBinder)`的辅助方法
-   AIDL 接口调用是直接方法调用，不应该想当然的认为它们在哪个线程中执行。如果是本地进程调用，那么接口会和调用者使用同一个线程执行；如果是远程调用，会由服务所在的进程的线程池派发一个线程来执行；如果使用oneway修饰，在远程调用中是非阻塞的，而在本地调用中依然是同步的。
-   AIDL 支持java 的基础类型、`String`、`Charsequence`、`List`、`Map`，除此之外的类型，都需要实现Parcebal接口，且必须使用import来声明，即使它们在同一个包
-   调用不保证会执行，所以从一开始设计的时候就应该考虑线程安全
-   为了避免ANR，应该将调用转移到单独的线程中执行
-   抛出的异常不会被调用者捕获（跨进程异常是不可取的）

###参考资料
-   [Services](https://developer.android.com/guide/components/services.html)
-   [Bound Services](https://developer.android.com/guide/components/bound-services.html)
-   [AIDL](https://developer.android.com/guide/components/aidl.html)

###参考代码
-   [MessengerService.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.java)
-   [MessengerServiceActivities.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.java)

