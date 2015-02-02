---
layout: post
title: "class文件从硬盘到内存的心路历程"
date: 2015-01-28 21:25
comments: true
categories: java笔记
keywords: 加载, 链接, 初始化
description: 本文主要描述了java类的加载、链接以及初始化的要点
---

##0x00 加载
-   **类的加载**本质上是由类加载器将字节码转换为`java.lang.Class`对象的过程
-   除了引导类加载器之外，所有的类加载器都有一个父类加载器
-   每一层的类加载器在收到加载请求之后，会将请求委派给父加载器执行；只有父加载器无法完成加载请求的时候，子加载器才尝试自己加载
-   这种双亲委托模型一方面避免了类的重复加载，另一方面还能防止Java 核心API**被子加载器替换**带来的隐患

##0x01 链接
-   链接分为三步：验证、准备和解析。其本质上是将字节码合入JVM运行时的过程。
-   **验证阶段**主要是确认字节码结构的合法性，一旦验证失败，JVM会抛出`java.lang.VerifyError`异常
-   **准备阶段**主要是将Java类的静态域初始化为默认值（并不执行任何代码）
-   **解析阶段**主要是确保所有引用的类能被正确的找到，这通常会导致引用类被加载

##0x02 初始化
该阶段的任务主要是初始化静态域，以及执行静态块代码。一个类被初始化之前，其父类会被先初始化。关于这一点，有2个需要注意的地方：

-   通过Java反射API也可能造成类的初始化
-   当访问一个类的静态域时，只有真正声明这个域的类才会被初始化。则，考虑如下代码：

```java
class Base {   
   static int value = 100;   
   static {       
      System.out.println("我会被输出.");    
   }
}

class A extends Base {   
   static {       
      System.out.println("我不会被输出"); 
   }
}

public class Test {   
   public static void main(String[] args) {       
      System.out.println(A.value);  
   }
}
```

如上所述，上述代码的运行结果为：

{% img center /images/201501/code_result.png %}

##0x03 参考资料

-   [深入探讨Java 类加载器][1]
-   [Java虚拟机类加载机制浅谈][2]


  [1]: http://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html
  [2]: http://computerdragon.blog.51cto.com/6235984/1223354