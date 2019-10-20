---
title: Java面试题
tags:
  - 面试
  - Java
categories:
  - 文章
toc: true
date: 2019-09-13 15:57:44
---

# 基础与框架

## String类能被继承吗，为什么?

> 不可以，因为String类有final修饰符，而final修饰的类是不能被继承的，实现细节不允许改变。

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence 
```

根据程序上下文环境，Java关键字final有“这是无法改变的”或者“终态的”含义，它可以修饰非抽象类、非抽象类成员方法和变量。你可能出于两种理解而需要阻止改变：设计或效率。 
　　final类不能被继承，没有子类，final类中的方法默认是final的。 
　　final方法不能被子类的方法覆盖，但可以被继承。 
　　final成员变量表示常量，只能被赋值一次，赋值后值不再改变。 
　　final不能用于修饰构造方法。 
　　注意：父类的private成员方法是不能被子类方法覆盖的，因此private类型的方法默认是final类型的。

如果一个类不允许其子类覆盖某个方法，则可以把这个方法声明为final方法。 
　　使用final方法的原因有二： 
　　第一、把方法锁定，防止任何继承类修改它的意义和实现。 
　　第二、高效。编译器在遇到调用final方法时候会转入内嵌机制，大大提高执行效率。（这点有待商榷，《Java编程思想》中对于这点存疑）

> 下面这段话摘自《Java编程思想》第四版第143页： 
“使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。”

关于String类，要了解常量池的概念
```java
String s = new String(“xyz”);  //创建了几个对象
```
答案： 1个或2个， 如果”xyz”已经存在于常量池中，则只在堆中创建”xyz”对象的一个拷贝，否则还要在常量池中在创建一份
```java
String s = "a"+"b"+"c"+"d"; //创建了几个对象
```
答案： 这个和JVM实现有关， 如果常量池为空，可能是1个也可能是7个等

## String，Stringbuffer，StringBuilder的区别？

1、用来处理字符串常用的类有3种：String、StringBuffer和StringBuilder
2、三者之间的区别：
都是final类，都不允许被继承；
String类长度是不可变的，StringBuffer和StringBuilder类长度是可以改变的；
StringBuffer类是线程安全的，StringBuilder不是线程安全的；

String 和 StringBuffer：
1、String类型和StringBuffer类型的主要性能区别：String是不可变的对象，因此每次在对String类进行改变的时候都会生成一个新的string对象，然后将指针指向新的string对象，所以经常要改变字符串长度的话不要使用string，因为每次生成对象都会对系统性能产生影响，特别是当内存中引用的对象多了以后，JVM的GC就会开始工作，性能就会降低；

2、使用StringBuffer类时，每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用，所以多数情况下推荐使用StringBuffer，特别是字符串对象经常要改变的情况；

3、在某些情况下，String对象的字符串拼接其实是被Java Compiler编译成了StringBuffer对象的拼接，所以这些时候String对象的速度并不会比StringBuffer对象慢，例如：

```java
String s1 = “This is only a” + “ simple” + “ test”;
StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”);
```
生成 String s1对象的速度并不比 StringBuffer慢。其实在Java Compiler里，自动做了如下转换：
```java
Java Compiler直接把上述第一条语句编译为：
String s2 = “This is only a”;
String s3 = “ simple”;
String s4 = “ test”;
String s1 = s2 + s3 + s4;
```
[传送门](https://www.jianshu.com/p/8c724dd28fa4)

## ArrayList和LinkedList有什么区别

ArrayList是实现了基于动态数组的结构，LinkedList则是基于实现链表的数据结构。

**数据的更新和查找**
ArrayList的所有数据是在同一个地址上,而LinkedList的每个数据都拥有自己的地址.所以在对数据进行查找的时候，由于LinkedList的每个数据地址不一样，get数据的时候ArrayList的速度会优于LinkedList，而更新数据的时候，虽然都是通过循环循环到指定节点修改数据，但LinkedList的查询速度已经是慢的，而且对于LinkedList而言，更新数据时不像ArrayList只需要找到对应下标更新就好，LinkedList需要修改指针，速率不言而喻

**数据的增加和删除**
对于数据的增加元素，ArrayList是通过移动该元素之后的元素位置，其后元素位置全部+1，所以耗时较长，而LinkedList只需要将该元素前的后续指针指向该元素并将该元素的后续指针指向之后的元素即可。与增加相同，删除元素时ArrayList需要将被删除元素之后的元素位置-1，而LinkedList只需要将之后的元素前置指针指向前一元素，前一元素的指针指向后一元素即可。当然，事实上，若是单一元素的增删，尤其是在List末端增删一个元素，二者效率不相上下。

[传送门](https://juejin.im/post/5adec492f265da0b9f3fea08#heading-2)

## 类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，他们的执行顺序

> 此题考察的是类加载器实例化时进行的操作步骤（加载–>连接->初始化）。 
父类静态变量、 
父类静态代码块、 
子类静态变量、 
子类静态代码块、 
父类非静态变量（父类实例成员变量）、 
父类构造函数、 
子类非静态变量（子类实例成员变量）、 
子类构造函数。 

[传送门](https://segmentfault.com/a/1190000004527951#articleHeader0)

## 用过哪些Map，都有什么区别，HashMap是线程安全的吗,并发下使用的Map是什么，他们内部原理分别是什么，比如hashcode，扩容等

> Hashtable,HashMap,ConcurrentHashMap

**线程不安全的HashMap**
因为多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。

**HashMap**
HashMap内部实现是一个桶数组，每个桶中存放着一个单链表的头结点。其中每个结点存储的是一个键值对整体（Entry），HashMap采用拉链法解决哈希冲突

[传送门](https://blog.csdn.net/u014532901/article/details/78936283)

**效率低下的HashTable容器**
     HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

**ConcurrentHashMap的锁分段技术**
     HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

[传送门](https://blog.csdn.net/qq_27093465/article/details/52279473)

> hashcode() 方法，在object类中定义如下：

```java
public native int hashCode();
```
native说明是一个本地方法，它的实现是根据本地机器相关的。当然我们可以在自己写的类中覆盖hashcode()方法，比如String、Integer、Double。。。。等等这些类都是覆盖了hashcode()方法的
例如String类中:就是以31为权，每一位为字符的ASCII值进行运算，用自然溢出来等效取模。(为什么取31?主要是因为31是一个奇质数，所以31i=32i-i=(i<<5)-i，这种位移与减法结合的计算相比一般的运算快很多).

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

## HashMap为什么get和set那么快，concurrentHashMap为什么能提高并发

> HashMap 底层是基于 数组 + 链表 组成的

[传送门](https://juejin.im/post/5b551e8df265da0f84562403#heading-9)

## 抽象类和接口的区别，类可以继承多个类么，接口可以继承多个接口么,类可以实现多个接口么

> 实现 抽象类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现。
[抽象类和接口的区别](https://www.jianshu.com/p/038f0b356e9a)

> 由于Java不支持多继承，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。

> 接口可以继承多个接口。
java类是单继承的。classB Extends classA
java接口可以多继承。Interface3 Extends Interface0, Interface1, interface……
不允许类多重继承的主要原因是，如果A同时继承B和C，而B和C同时有一个D方法，A如何决定该继承那一个呢？
但接口不存在这样的问题，接口全都是抽象方法继承谁都无所谓，所以接口可以继承多个接口。

## 什么情况下会发生栈内存溢出

**方法递归调用产生这种结果**
> 栈是线程私有的，他的生命周期与线程相同，每个方法在执行的时候都会创建一个栈帧，用来存储局部变量表，操作数栈，动态链接，方法出口灯信息。局部变量表又包含基本数据类型，对象引用类型（局部变量表编译器完成，运行期间不会变化）

所以我们可以理解为栈溢出就是方法执行是创建的栈帧超过了栈的深度。那么最有可能的就是方法递归调用产生这种结果。栈溢出(StackOverflowError)


## 什么是nio，原理

> NIO是为了弥补传统I/O工作模式的不足而研发的，NIO的工具包提出了基于Selector（选择器）、Buffer（缓冲区）、Channel（通道）的新模式；Selector（选择器）、可选择的Channel（通道）和SelectionKey（选择键）配合起来使用，可以实现并发的非阻塞型I/O能力。

NIO的工作原理是什么？

　　在并发型服务器程序中使用NIO，实际上是通过网络事件驱动模型实现的。我们应用Select机制，不用为每一个客户端连接新启线程处理，而是将其注册到特定的Selector对象上，这就可以在单线程中利用Selector对象管理大量并发的网络连接，更好的利用了系统资源；采用非阻塞I/O的通信方式，不要求阻塞等待I/O操作完成即可返回，从而减少了管理I/O连接导致的系统开销，大幅度提高了系统性能。

　　当有读或写等注册事件发生时，可以从Selector中获得相应的SelectionKey，从SelectionKey中可以找到发生的事件和该事件所发生的具体的SelectableChannel，以获得客户端发送过来的数据。由于在非阻塞网络I/O中采用了事件触发机制，处理程序可以得到系统的主动通知，从而可以实现底层网络I/O无阻塞、流畅地读写，而不像在原来的阻塞模式下处理程序需要不断循环等待。使用NIO，可以编写出性能更好、更易扩展的并发型服务器程序。

　　并发型服务器程序的实现代码：应用NIO工具包，基于非阻塞网络I/O设计的并发型服务器程序与以往基于阻塞I/O的实现程序有很大不同，在使用非阻塞网络I/O的情况下，程序读取数据和写入数据的时机不是由程序员控制的，而是Selector决定的。

　　使用非阻塞型I/O进行并发型服务器程序设计分三个部分：1. 向Selector对象注册感兴趣的事件；2.从Selector中获取所感兴趣的事件；3. 根据不同的事件进行相应的处理。

　　在进行并发型服务器程序设计时，通过合理地使用NIO工具包，就可以达到一个或者几个Socket线程就可以处理N多个Socket的连接，大大降低我们对服务器程序的预算压力。同时我们利用它更好地提高系统的性能，使我们的工作得到更加有效地开展。

[传送门](https://www.cnblogs.com/johnvajicic/archive/2013/04/18/3028675.html)

## 反射中，Class.forName和ClassLoader区别

Java中Class.forName和classloader都可以用来对类进行加载。

- Class.forName(“className”);
        其实这种方法调运的是：Class.forName(className, true, ClassLoader.getCallerClassLoader())方法
        参数一：className，需要加载的类的名称。
        参数二：true，是否对class进行初始化（需要initialize）
        参数三：classLoader，对应的类加载器

- ClassLoader.laodClass(“className”);
        其实这种方法调运的是：ClassLoader.loadClass(name, false)方法
        参数一：name,需要加载的类的名称
        参数二：false，这个类加载以后是否需要去连接（不需要linking）

可见Class.forName除了将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块。

而classloader只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容，只有在newInstance才会去执行static块。

[传送门](https://blog.csdn.net/u013308490/article/details/87809824)

## tomcat结构，类加载器流程

> Tomcat 的总体结构

![image.png](/images/2019/09/13/12f00090-d605-11e9-a6a7-0b204ec53b69.png)

从上图中可以看出 Tomcat 的心脏是两个组件：Connector 和 Container，关于这两个组件将在后面详细介绍。Connector 组件是可以被替换，这样可以提供给服务器设计者更多的选择，因为这个组件是如此重要，不仅跟服务器的设计的本身，而且和不同的应用场景也十分相关，所以一个 Container 可以选择对应多个 Connector。多个 Connector 和一个 Container 就形成了一个 Service，Service 的概念大家都很熟悉了，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了。所以整个 Tomcat 的生命周期由 Server 控制。

**什么是类加载器？**
虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块称为“类加载器”。

[传送门](https://www.jianshu.com/p/69c4526b843d)

## 讲讲Spring事务的传播属性,AOP原理，动态代理与cglib实现的区别，AOP有哪几种实现方式

## Spring的beanFactory和factoryBean的区别


## Spring加载流程


## Spring如何管理事务的


# 多线程

## 线城池的最大线程数目根据什么确定
## 多线程的几种实现方式，什么是线程安全，什么是重排序
## volatile的原理，作用，能代替锁么
## sleep和wait的区别，以及wait的实现原理
## Lock与synchronized 的区别，synchronized 的原理，什么是自旋锁，偏向锁，轻量级锁，什么叫可重入锁，什么叫公平锁和非公平锁
## 用过哪些原子类，他们的参数以及原理是什么
## 用过哪些线程池，他们的原理简单概括下，构造函数的各个参数的含义，比如coreSize，maxsize等
## 有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有10个线程同时调用它，如何做到。
## spring的controller是单例还是多例，怎么保证并发的安全
## 用三个线程按顺序循环打印abc三个字母，比如abcabcabc
## ThreadLocal用过么，原理是什么，用的时候要注意什么
## 如果让你实现一个并发安全的链表，你会怎么做

# JVM相关

## jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代，几种主要的jvm参数等
## 你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms
## 当出现了内存溢出，你怎么排错
## JVM内存模型的相关知识了解多少
## 简单说说你了解的类加载器
## JAVA的反射机制

# 网络

## http1.0和http1.1有什么区别
## TCP三次握手和四次挥手的流程，为什么断开连接要4次,如果握手只有两次，会出现什么
## TIME_WAIT和CLOSE_WAIT的区别
## 说说你知道的几种HTTP响应码
## 当你用浏览器打开一个链接的时候，计算机做了哪些工作步骤
## Linux下IO模型有几种，各自的含义是什么
## TCP/IP如何保证可靠性，数据包有哪些数据组成
## 架构设计与分布式：
## tomcat如何调优，各种参数的意义
## 常见的缓存策略有哪些，你们项目中用到了什么缓存系统，如何设计的，Redis的使用要注意什么，持久化方式，内存设置，集群，淘汰策略等
## 如何防止缓存雪崩12.用java自己实现一个LRU
## 分布式集群下如何做到唯一序列号
## 设计一个秒杀系统，30分钟没付款就自动关闭交易
## 如何做一个分布式锁
## 用过哪些MQ，怎么用的，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗
## MQ系统的数据如何保证不丢失
## 分布式事务的原理，如何使用分布式事务
## 什么是一致性hash
## 什么是restful，讲讲你理解的restful
## 如何设计建立和保持100w的长连接？
## 解释什么是MESI协议(缓存一致性)
## 说说你知道的几种HASH算法，简单的也可以
## 什么是paxos算法
## redis和memcached 的内存管理的区别
## 一个在线文档系统，文档可以被编辑，如何防止多人同时对同一份文档进行编辑更新

# 算法

## 10亿个数字里里面找最小的10个
## 有1亿个数字，其中有2个是重复的，快速找到它，时间和空间要最优
## 2亿个随机生成的无序整数,找出中间大小的值
## 遍历二叉树

# 数据库
## 数据库隔离级别有哪些，各自的含义是什么，MYsql默认的隔离级别是是什么，各个存储引擎优缺点
## 高并发下，如何做到安全的修改同一行数据，乐观锁和悲观锁是什么，INNODB的行级锁有哪2种，解释其含义
## SQL优化的一般步骤是什么，怎么看执行计划，如何理解其中各个字段的含义，索引的原理？
## 数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁
## MYsql的索引实现方式
## 聚集索引和非聚集索引的区别
## 数据库中 BTREE和B+tree区别