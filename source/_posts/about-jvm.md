title: Java Virtual Machine
date: 2015-05-01 13:44:09
tags: [Java, JVM]
---

![The internal architecture of the Java virtual machine.](/blogs/img/about-jvm-architecture.jpg)
上图为JVM框架，图片来源：[Inside the Java 2 Virtual Machine 2nd Edition](http://www.artima.com/insidejvm/ed2/jvm2.html)

<!-- more -->

虽然这*Inside the Java 2 Virtual Machine 2nd Edition*本书([PDF](http://mihaimoldovan.com/download/Inside-Java-Virtual-Machine.pdf))是在2000年6月出版的，已经很古老了，但从其给出的JVM结构图来看，JVM整体框架在这15年里几乎没有任何变化。维基百科给出的基于Java SE 7 Edition的[JVM框架](http://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/JvmSpec7.png/400px-JvmSpec7.png)几乎与这个一模一样。[最新消息](https://anturis.com/blog/java-virtual-machine-the-essential-guide/)称JVM已将方法区(Method Area)移除了，将其放在了操作系统层面。单从官方给出的JVM说明[文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.4)来看，并未移除该部分的内容，也许消息发布者是针对某一个具体的JVM实现来说的。

抛开年代，就其内容来说，该书主要从设计文档(the abstract specification)、具体实现(a concrete implementation)和运行实例(a runtime instance)三个角度来讲述JVM，并且涉及操作系统、编译等知识，讲得很透彻。

之所以要看JVM相关的东西，主要是听周围的人都在谈论这个东西，所以想一探究竟。我结合自身需求，主要看了本书的第5、8、9、20章，主要浏览了Java虚拟机的框架、类加载器、垃圾回收和线程同步的内容。

# Java Virtual Machine
从框架设计来看，JVM总体有5部分内容：
* 类加载器子系统：启动类加载器(bootstrap)、用户自定义加载器等
* 运行时数据区
* 执行引擎：解释执行、即时(Just-In-Time)编译、Java线程模型等
* 本地方法接口：JNI等
* 本地方法库

运行时数据区又分为5部分：
* 方法区(method area)：类型信息、常量等
* 堆(heap)：只放对象和数组
* 栈(Java Stacks)：各线程的方法栈等，一般称为Java虚拟机栈
* 程序计数器(PC registers)：各线程的下一条执行语句
* 本地方法栈(native method stacks)：非Java语言方法栈

![Method Area V.S. Heap](/blogs/img/about-jvm-method-area-heap.jpg)
上图为JVM中方法区与堆区的区别，图片来源：[Inside the Java 2 Virtual Machine 2nd Edition](http://www.artima.com/insidejvm/ed2/jvm2.html)

![Relations between PC Registers, Java Stacks and Native Method Stacks](/blogs/img/about-jvm-pc-stack-native-method-stack-relations.jpg)
上图为JVM中程序计数器、Java虚拟机栈、本地方法栈之间的关系，图片来源：[Inside the Java 2 Virtual Machine 2nd Edition](http://www.artima.com/insidejvm/ed2/jvm2.html)

![Invoke Java and Native Methods in JVM](/blogs/img/about-jvm-invokes-java-and-native-methods.jpg)
上图为JVM中调用Java方法与本地方法之间的区别与联系，图片来源：[Inside the Java 2 Virtual Machine 2nd Edition](http://www.artima.com/insidejvm/ed2/jvm9.html)

# Garbage Collection
![GC Tuning in JVM](/blogs/img/about-jvm-gc-tuning.png)
图片来源[Java SE 6 GC Tuning](http://www.oracle.com/technetwork/java/javase/gc-tuning-6-140523.html)，未找到Java 7相关的资料，官方只有简单的[介绍](http://docs.oracle.com/javase/7/docs/technotes/guides/vm/)。
垃圾回收是针对堆和方法区来讲的，主要思想是：内存快用完了，清除那些没用的对象吧。回收的算法有很多种，如（以下方法来自[Inside the Java 2 Virtual Machine 2nd Edition](http://www.artima.com/insidejvm/ed2/jvm9.html)）：
* 引用计数法
* 轨迹跟踪法
* 压缩拷贝法
* 停止-拷贝法：Thinking in Java中提到的方法，比较流行，缺点是清理时所有线程（除了清理垃圾的GC线程）都得停止运行
* 自适应法
* 火车法

不同的JVM实现可能会采用不同的方法，下图是一种流行的停止-拷贝法，对堆区的整理方式分三个层级，详细参阅[Java SE 6 GC Tuning](http://www.oracle.com/technetwork/java/javase/gc-tuning-6-140523.html)或[Java SE 8 GC Tuning](http://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/index.html)：
* Eden Pool是新生代，满了之后将垃圾清空留下来的存入Survivor Pool存活区
* Survivor Pool是存活区，每次有新的对象进入，原有的存活对象年龄加1，满了之后将年龄超过一定限度的放入Tenured Pool终身存活区。
* Tenured Pool终身存活区，满了之后运行GC清空无用对象。

需要注意的是方法区虽是永久代，但并不代表不被清理，它只是相对堆区更为稳定而已，满了之后同样由GC线程清理。

![A Popular GC Implementation](/blogs/img/about-jvm-architecture-with-gc.png)
上图为一种流行的GC实现方式，图片来源[Java虚拟机：基本指南](https://anturis.com/blog/java-virtual-machine-the-essential-guide/)

# 同步与协作
JVM中堆和方法区被所有线程共享的，为保证数据同步和多线程之间的协作，JVM提供了一种机制叫做monitor。
对象锁：JVM中monitor的实际实现形式，就是对象锁，JVM中的锁都是对象锁。
类锁：实际上是以对象锁的形式实现的，因为JVM在加载一个类文件时会创建一个java.lang.Class实例，对一个类加锁时，实际上是对那个类的Class实例对象加锁。

# 监控JVM
[Anturis Console-JVM monitoring](https://anturis.com/jvm-monitoring/)