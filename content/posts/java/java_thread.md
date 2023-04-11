---
title: "《深入浅出Java多线程》--原理篇"
date: 2023-04-11T11:06:29+08:00
draft: false
categories: ["Java"]
tags: ["Java"]
---

## 前言
在本文开篇之前，先介绍几个概念：
* **内存可见性**：指的是线程之间的可见性，当一个线程修改了共享变量时，另一个线程可以读取到这个修改后的值。
* **重排序**：为优化程序性能，对原有的指令执行顺序进行优化重新排序。重排序可能发生在多个阶段，例如编译重排序、CPU重排序、内存重排序等。
* **happens-before规则**：是一个给程序员使用的规则，只要程序员在写代码的时候遵循happens-before规则，JVM就能保证指令在多线程之间的顺序性符合程序员的预期。
## 一、Java内存模型基础知识

### 现代计算机的内存模型

早期计算机中cpu和内存的速度是差不多的，但在现代计算机中，cpu的指令速度远超内存的存取速度,由于计算机的存储设备与处理器的运算速度有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（Cache）来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了。

![现代计算机的内存模型](/images/java_thread_1.webp)

现代的处理器使用写缓冲区临时保存向内存写入的数据。写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，减少对内存总线的占用。虽然写缓冲区有这么多好处，但**每个处理器上的写缓冲区，仅仅对它所在的处理器可见**。这个特性会对内存操作的执行顺序产生重要的影响：**处理器对内存的读/写操作的执行顺序，不一定与内存实际发生的读/写操作顺序一致！**

为了具体说明，请看下面示例：

![](/images/java_thread_2.webp)

![](/images/java_thread_3.webp)

处理器A和处理器B按程序的顺序并行执行内存访问，最终可能得到x=y=0的结果。
处理器A和处理器B可以同时把共享变量写入自己的写缓冲区（A1，B1），然后从内存中读取另一个共享变量（A2，B2），最后才把自己写缓存区中保存的脏数据刷新到内存中（A3，B3）。
* 当以这种时序执行时，主内存中a、b变量并没有被及时的刷新到主内存中，处理器A和处理器B正则执行的程序没有及时拿到修改后的值，导致了内存可见性问题，程序得到x=y=0的结果。
* 从内存操作实际发生的顺序来看，直到处理器A执行A3来刷新自己的写缓存区，写操作A1才算真正执行了。虽然处理器A执行内存操作的顺序为：A1→A2，但内存操作实际发生的顺序却是A2→A1。

### Java内存模型的抽象
Java内存模型（简称JMM）定义了Java 虚拟机(JVM)在计算机内存(RAM)中的工作方式。JVM是整个计算机虚拟模型，所以JMM是隶属于JVM的。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。**本地内存是JMM的一个抽象概念，并不真实存在**。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

![](/images/java_thread_4.webp)

从上图来看，线程A与线程B通讯，必须经历下面2个阶段：
* 线程A更新本地内存变量A，刷新到主内存中的共享变量
* 线程B读取线程A刷新到主内存的共享变量

这样JMM就通过控制主内存与每个线程的本地内存之间的交互，来提供内存可见性保证。
## 二、JVM对JMM的实现

先谈一下运行时数据区，下面这张图相信大家一点都不陌生：

![Java运行时数据区](/images/java_thread_5.webp)

对于每一个线程来说，栈都是私有的，而堆是共有的。也就是说在栈中的变量（局部变量、方法定义参数、异常处理器参数）不会在线程之间共享，也就不会有内存可见性的问题，也不受内存模型的影响。而在堆中的变量是共享的，本文称为共享变量。

**在JVM内部，Java内存模型把内存分成了两部分：线程栈区和堆区**

JVM中运行的每个线程都拥有自己的线程栈，线程栈包含了当前线程执行的方法调用相关信息，我们也把它称作调用栈。随着代码的不断执行，调用栈会不断变化。
所有原始类型(boolean,byte,short,char,int,long,float,double)的局部变量都直接保存在线程栈当中，对于它们的值各个线程之间都是独立的。对于原始类型的局部变量，一个线程可以传递一个副本给另一个线程，当它们之间是无法共享的。
堆区包含了Java应用创建的所有对象信息，不管对象是哪个线程创建的，其中的对象包括原始类型的封装类（如Byte、Integer、Long等等）。不管对象是属于一个成员变量还是方法中的局部变量，它都会被存储在堆区。
一个局部变量如果是原始类型，那么它会被完全存储到栈区。 一个局部变量也有可能是一个对象的引用，这种情况下，这个本地引用会被存储到栈中，但是对象本身仍然存储在堆区。
对于一个对象的成员方法，这些方法中包含局部变量，仍需要存储在栈区，即使它们所属的对象在堆区。 对于一个对象的成员变量，不管它是原始类型还是包装类型，都会被存储到堆区。Static类型的变量以及类本身相关信息都会随着类本身存储在堆区。

![](/images/java_thread_6.webp)

## 三、重排序
计算机在执行程序时，为了提升性能，编译器和处理器常常会对指令做重排。

**为什么指令重排序可以提升性能？**
简单地说，每一个指令都会包含多个步骤，每个步骤可能使用不同的硬件。因此，流水线技术产生了，它的原理是指令1还没有执行完，就可以开始执行指令2，而不用等到指令1执行结束之后再执行指令2，这样就大大提高了效率。
但是，流水线技术最害怕中断，恢复中断的代价是比较大的，所以我们要想尽办法不让流水线中断。**指令重排就是减少中断的一种技术**。

指令重排一般分为以下三种：
* 编译器优化重排
  编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
* 指令并行重排
  现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性(即后一个执行的语句无需依赖前面执行的语句的结果)，处理器可以改变语句对应的机器指令的执行顺序。
* 内存系统重排
  由于处理器使用缓存和读写缓存冲区，这使得加载(load)和存储(store)操作看上去可能是在乱序执行，因为三级缓存的存在，导致内存与缓存的数据同步存在时间差。
  ![](/images/java_thread_7.webp)

## 四、 顺序一致性模型与JMM的保证
* **数据竞争**：在一个线程中写一个变量，在另一个线程读同一个变量，并且写 和读没有通过同步来排序。
* **顺序一致性**: 即程序的执行结果和该程序在顺序一致性模型中执行的结果相同

如果程序中包含了数据竞争，那么运行的结果往往充满了不确定性，比如读发生在了写之前，可能就会读到错误的值；如果一个线程程序能够正确同步，那么就不存在数据竞争。

Java内存模型（JMM）对于正确同步多线程程序的内存一致性做了以下保证：
如果程序是正确同步的，程序的执行将具有顺序一致性。这里的同步包括了使用 volatile 、 final 、 synchronized 等关键字来实现多线程下的同步。

**顺序一致性模型**

顺序一致性内存模型是一个理想化的理论参考模型，它为程序员提供了极强的内存可见性保证。有以下2大特征：
* 一个线程中的所有操作必须按照程序的顺序（即Java代码的顺序）来执行
* 不管程序是否同步，所有线程都只能看到一个单一的操作执行顺序。即在顺序一致性模型中，每个操作必须是原子性的，且立刻对所有线程可见。

为了理解这两个特性，我们举个例子，假设有两个线程A和B并发执行，线程A有3个操作，他们在程序中的顺序是A1->A2->A3，线程B也有3个操作，B1->B2->B3。
假设正确使用了同步，A线程的3个操作执行后释放锁，B线程获取同一个锁。那么在顺序一致性模型中的执行效果如下所示：

![](/images/java_thread_8.webp)

假设没有使用同步，那么在顺序一致性模型中的执行效果如下所示：

![](/images/java_thread_9.webp)

操作的执行整体上无序，但是两个线程都只能看到这个执行顺序。之所以可以得到这个保证，是因为顺序一致性模型中的**每个操作必须对即对任意线程可见**。

**但是JMM没有这样的保证**，比如，在当前线程把写过的数据缓存在本地内存中，在没有刷新到主内存之前，这 个写操作仅对当前线程可见；从其他线程的角度来观察，这个写操作根本没有被当前线程所执行。只有当前线程把本地内存中写过的数据刷新到主内存之后，这个写操作才对其他线程可见。在这种情况下，当前线程和其他线程看到的执行顺序是不一样的。

**JMM中同步程序的顺序一致性效果**

在顺序一致性模型中，所有操作完全按照程序的顺序串行执行。但是JMM中，临界区内（同步块或同步方法中）的代码可以发生重排序（但不允许临界区内的代码 “逃逸”到临界区之外，因为会破坏锁的内存语义）。

虽然线程A在临界区做了重排序，但是因为锁的特性，线程B无法观察到线程A在临界区的重排序。这种重排序既提高了执行效率，又没有改变程序的执行结果。

同时，JMM会在退出临界区和进行临界区做特殊的处理，使得在临界区内程序获得 与顺序一致性模型相同的内存视图。

![](/images/java_thread_10.webp)

由此可见，JMM的具体实现方针是：在不改变（正确同步的）程序执行结果的前提下，尽量为编译期和处理器的优化打开方便之门。

## 五、volatile
在Java中，volatile关键字有特殊的内存语义。volatile主要有以下两个功能：
* 保证变量的内存可见性
* 禁止volatile变量与普通变量重排序（JSR133提出）

### volatile变量内存可见性
以一段示例代码开始：
```java
public class VolatileExample {
   int a = 0;
   volatile boolean flag = false;
   public void writer() {
     a = 1; // step 1
     flag = true; // step 2
   }
   public void reader() {
     if (flag) { // step 3
      System.out.println(a); // step 4
     }
   }
}
```
在这段代码里，我们使用 volatile 关键字修饰了一个 boolean 类型的变量 flag 。
所谓内存可见性，指的是当一个线程对 volatile 修饰的变量进行写操作（比如step 2）时，**JMM会立即把该线程对应的本地内存中的共享变量的值刷新到主内存**；
当一个线程对 volatile 修饰的变量进行读操作（比如step 3）时，**JMM会把立即该线程对应的本地内存置为无效，从主内存中读取共享变量的值**。
![](/images/java_thread_11.webp)

### 禁止重排序
为了提供一种比锁更轻量级的线程间的通信机制，JSR-133专家组决定增强
volatile的内存语义：严格限制编译器和处理器对volatile变量与普通变量的重排序。 编译器还好说，JVM是怎么还能限制处理器的重排序的呢？它是通过**内存屏障**来实现的。

什么是内存屏障？硬件层面，内存屏障分两种：读屏障（Load Barrier）和写屏障 （Store Barrier）。内存屏障有两个作用：
*  阻止屏障两侧的指令重排序；
*  强制把写缓冲区/高速缓存中的脏数据等写回主内存，或者让缓存中相应的数据
   失效。

编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。编译器选择了一个比较保守的JMM内存屏障插入策略，这样可以保证在任何处理器平台，任何程序中都能得到正确的volatile内存语义。这个策略是：
* 在每个volatile写操作前插入一个StoreStore屏障；
* 在每个volatile写操作后插入一个StoreLoad屏障；
* 在每个volatile读操作后插入一个LoadLoad屏障；
* 在每个volatile读操作后再插入一个LoadStore屏障

![](/images/java_thread_12.webp)

从volatile的内存语义上来看，volatile可以保证内存可见性且禁止重排序。

在保证内存可见性这一点上，volatile有着与锁相同的内存语义，所以可以作为一个“轻量级”的锁来使用。但由于volatile仅仅保证对单个volatile变量的读/写具有原子 性，而锁可以保证整个临界区代码的执行具有原子性。所以**在功能上，锁比volatile更强大；在性能上，volatile更有优势**。
## 六、Synchronized关键字
说到锁，我们通常会谈到 synchronized 这个关键字。它翻译成中文就是“同步”的意思。
我们通常使用synchronized 关键字来给一段代码或一个方法上锁。它通常有以下
三种形式：
```java
// 关键字在实例⽅法上，锁为当前实例
public synchronized void instanceLock() {
 // code
}
// 关键字在静态⽅法上，锁为当前Class对象
public static synchronized void classLock() {
 // code
}
// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
 Object o = new Object();
 synchronized (o) {
 // code
 }
}

```
我们这里介绍一下“临界区”的概念。所谓“临界区”，指的是某一块代码区域，它同一时刻只能由一个线程执行。在上面的例子中，如果 synchronized 关键字在方法上，那临界区就是整个方法内部。而如果是使用synchronized代码块，那临界区就指的是代码块内部的区域。

Java 6 为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁“。 在Java 6 以前，所有的锁都是”重量级“锁。所以在Java 6 及其以后，一个对象其实有四种锁状态，它们级别由低到高依次是：
* 无锁状态
* 偏向锁状态
* 轻量级锁状态
* 重量级锁状态

**Java对象头**
前面我们提到，Java的锁都是基于对象的。首先我们来看看一个对象的“锁”的信息 是存放在什么地方的。 每个Java对象都有对象头。如果是非数组类型，则用2个字宽来存储对象头，如果是数组，则会用3个字宽来存储对象头。在32位处理器中，一个字宽是32位；在64位虚拟机中，一个字宽是64位。对象头的内容如下表：

长度|内容|说明
:--|:--|:--
32/64bit|Mark Word|存储对象的hashCode或锁信息等
32/64bit|Class Metadata Address|存储到对象类型数据的指针
32/64bit|Array length|数组的长度（如果是数组）

我们主要来看看Mark Word的格式：
锁状态|29bit或61bit|1bit是都是偏向锁？|2bit锁标记位
:--|:--|:--|:--
无锁||0|01
偏向锁|线程ID|1|01
轻量级锁|指向栈中锁记录的指针|此时这一位不用于标识偏向锁|00
重量级锁|指向互斥量（重量级锁）的指针|此时这一位不用于标识偏向锁|10
GC标记||此时这一位不用于标识偏向锁|11

**偏向锁**
偏向锁会偏向于第一个访问锁的线程，如果在接下来的运行过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将永远不需要触发同步。也就是说，**偏向锁在资源无竞争情况下消除了同步语句，连CAS操作都不做了，提高了程序的运行性能**。

偏向锁实现原理：
一个线程在第一次进入同步块时，会在对象头和栈帧中的锁记录里存储锁的偏向的线程ID。当下次该线程进入这个同步块时，会去检查锁的Mark Word里面是不是放的自己的线程ID。
如果是，表明该线程已经获得了锁，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁 ；如果不是，就代表有另一个线程来竞争这个偏向锁。这个时候会尝试使用CAS来替换Mark Word里面的线程ID为新线程的ID，这个时候要分两种情况：
* 成功，表示之前的线程不存在了， Mark Word里面的线程ID为新线程的ID，锁不会升级，仍然为偏向锁；
* 失败，表示之前的线程仍然存在，那么暂停之前的线程，设置偏向锁标识为0，并设置锁标志位为00，升级为轻量级锁，会按照轻量级锁的方式进行竞争锁。

**撤销偏量锁**
偏向锁使用了一种**等到竞争出现才释放锁的机制**，所以当其他线程尝试竞争偏向锁时， 持有偏向锁的线程才会释放锁。
偏向锁升级成轻量级锁时，会暂停拥有偏向锁的线程，重置偏向锁标识，这个过程 看起来容易，实则开销还是很大的，大概的过程如下：
*  在一个安全点停止拥有锁的线程。
*  遍历线程栈，如果存在锁记录的话，需要修复锁记录和Mark Word，使其变成无锁状态。
* 唤醒被停止的线程，将当前锁升级成轻量级锁
  所以，如果应用程序里所有的锁通常出于竞争状态，那么偏向锁就会是一种累赘， 对于这种情况，我们可以一开始就把偏向锁这个默认功能给关闭：
```
-XX:UseBiasedLocking=false
```
![偏量锁的获得和撤销](/images/java_thread_13.webp)

**轻量级锁的加锁与释放**
JVM会为每个线程在当前线程的栈帧中创建用于存储锁记录的空间，我们称为
Displaced Mark Word。**如果一个线程获得锁的时候发现是轻量级锁（锁标记位为00），会把锁的Mark Word复制到自己的Displaced Mark Word里面**。

然后线程尝试用CAS将锁的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示Mark Word已经被替换成了其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使用自旋来获取锁。

自旋也不是一直进行下去的，如果自旋到一定程度（和JVM、操作系统相关），依然没有获取到锁，称为**自旋失败，那么这个线程会阻塞。同时这个锁就会升级成重量级锁**。

在轻量级锁的同步体被执行完之后，当前线程会进行**轻量级锁的释放**，在释放锁时，当前线程会使用CAS操作将Displaced Mark Word的内容复制回锁的Mark Word里面。如果没有发生竞争，那么这个复制的操作会成功。如果有其他线程因为自旋多次导致轻量级锁升级成了重量级锁，那么CAS操作会失败，此时会释放锁并唤醒被阻塞的线程。

![](/images/java_thread_14.webp)

**锁升级流程总结**

每一个线程在准备获取共享资源时：
* 第一步，检查MarkWord里面是不是放的自己的ThreadId ,如果是，表示当前线程是处于 “偏向锁” 。
* 第二步，如果MarkWord不是自己的ThreadId，锁升级，这时候，用CAS来执行切换，新的线程根据MarkWord里面现有的ThreadId，通知之前线程暂停，之前线程将Markword的内容置为空。恢复之前挂起的线程，偏向锁升级为轻量锁。
* 第三步，两个线程都把锁对象的HashCode复制到自己新建的用于存储锁的记录空 间，接着开始通过CAS操作， 把锁对象的MarKword的内容修改为自己新建的记录空间的地址的方式竞争MarkWord。
* 第四步，第三步中成功执行CAS的获得资源，失败的则进入自旋 。
* 第五步，自旋的线程在自旋过程中，成功获得资源(即之前获的资源的线程执行完成并释放了共享资源)，则整个状态依然处于轻量级锁的状态，如果自旋失败 。
* 第六步，进行重量级锁的状态，这个时候，自旋的线程进行阻塞，等待之前线程执 行完成并唤醒自己

## 七、线程池原理

### 为什么要用线程池？

使用线程池主要有以下三个原因：

* 创建/销毁线程需要消耗系统资源，线程池可以复用已创建的线程
* 控制并发的数量。并发数量过多，可能会导致资源消耗过多，从而造成服务器崩溃
* 可以对线程做统一管理

### ThreadPoolExecutor

Java中的线程池顶层接口是 Executor 接口， ThreadPoolExecutor 是这个接口的实现类。我们来看看这个类的构造函数

```java
// 五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue)
// 六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory)
// 六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 RejectedExecutionHandler handler)
// 七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory,
 RejectedExecutionHandler handler)  
```
主要的几个参数介绍说明：

* corePoolSize：该线程池的核心线程数
* maximumPoolSize：改线程池最大的线程数
* keepAliveTime：非核心线程的闲置超时时间
* unit：超时时间单位
* workQueue：阻塞队列

### 线程池主要的处理流程

* 线程总数量 < corePoolSize，无论线程是否空闲，都会新建一个核心线程执行任务（让核心线程数量快速达到corePoolSize，在核心线程数量 <corePoolSize时）。**注意，这一步需要获得全局锁。**
* 线程总数量 >= corePoolSize时，新来的线程任务会进入阻塞队列中等待，然后空闲的核心线程会依次去阻塞队列中取任务来执行（体现了线程**复用**）。
* 当缓存队列满了，说明这个时候任务已经多到爆棚，需要一些“临时工”来执行这些任务了。于是会创建非核心线程去执行这个任务。**注意，这一步需要获得全局锁**。
*  缓存队列满了， 且总线程数达到了maximumPoolSize，则会采取**拒绝策略进行处理**

   ![](/images/java_thread_15.webp)

   **四种常见的线程池**
   Executors 类中提供的一个静态方法来创建线程池。大家到了这一步，如果看懂了前面讲的 ThreadPoolExecutor 构造方法中各种参数的意义，那么一看到 Executors 类中提供的线程池的源码就应该知道这个线程池是干嘛的

* **newCachedThreadPool**
```java
public static ExecutorService newCachedThreadPool() {
     return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
     60L, TimeUnit.SECONDS,
     new SynchronousQueue<Runnable>());
}
```

由于这边核心线程数为0，所以newCachedThreadPool只会创建非核心线程，并且这边指定了非核心线程闲置超时时间为60S,线程最大数为Integer.MAX_VALUE。使用这个线程的静态方式适用于处理很多**短时间的任务**，复用率很高，而且因为超时时间为60S，所以也**不会占用太多的资源**。

* **newFixedThreadPool**

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
   return new ThreadPoolExecutor(nThreads, nThreads,
   0L, TimeUnit.MILLISECONDS,
   new LinkedBlockingQueue<Runnable>());
}
```

可以看到这边的核心线程数和最大线程数一致，所以newFixedThreadPool只会创建核心线程，因为LinkedBlockingQueue队列的默认大小也是Integer.MAX_VALUE，所以当任务数大于设定的核心线程数时，会把线程放入阻塞队列中，直到核心线程空闲，才会从阻塞队列拿任务到核心线程中执行。这种线程池适用于处理**多个长时间的任务**，但是尽量少用，因为如果队列中没有任务可取，线程会一直阻塞在LinkedBlockingQueue.take() ，线程不会被回收，**占用资源多**

* **newSingleThreadExecutor**

```java
public static ExecutorService newSingleThreadExecutor() {
   return new FinalizableDelegatedExecutorService
   (new ThreadPoolExecutor(1, 1,
   0L, TimeUnit.MILLISECONDS,
   new LinkedBlockingQueue<Runnable>()));
}
```

从构造方法参数可以看出，基本和newFixedThreadPool一致，唯一的区别就是最大线程数为1，意味着这种线程池每次只会在核心线程池里执行1个任务，并且以FIFO的模式运行。

* **newScheduledThreadPool**
  创建一个定长线程池，支持定时及周期性任务执行

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize
   return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
   super(corePoolSize, Integer.MAX_VALUE,
   DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
   new DelayedWorkQueue());
}
```
## 八、锁接口和类

**synchronized的不足**
* 如果临界区是只读操作，其实可以多线程一起执行，但使用synchronized的话，同一时间只能有1个线程执行
* synchronized无法知道线程有没有成功获取到锁
* 使用synchronized，如果临界区因为IO或者sleep方法等原因阻塞了，而当前线程又没有释放锁，就会导致所有线程等待

**锁的几种分类**

* 可重入锁和非可重入锁
  synchronized关键字就是使用的重入锁。比如说，你在一个synchronized实例方法 里面调用另一个本实例的synchronized实例方法，它可以重新进入这个锁，不会出现任何异常。 如果我们自己在继承AQS实现同步器的时候，没有考虑到占有锁的线程再次获取锁的场景，可能就会导致线程阻塞，那这个就是一个“非可重入锁”。
* 公平锁与非公平锁
  如果对一个锁来说，先对锁获取请求的线程一定会先被满足，后对锁获取请求的线程后被满足，那这个锁就是公平的。反之，那就是不公平的。
  一般情况下，非公平锁能提升一定的效率。但是**非公平锁可能会发生线程饥饿**（有一些线程长时间得不到锁）的情况。
* 读写锁和排它锁
  我们前面讲到的synchronized用的锁和ReentrantLock，其实都是“排它锁”。也就是 说，这些锁在**同一时刻只允许一个线程进行访问**。
  而读写锁可以再**同一时刻允许多个读线程访问**。

**ReentrantLock**

ReentrantLock是一个非抽象类，它是Lock接口的JDK默认实现，实现了锁的基本功能。从名字上看，它是一个”可重入“锁，从源码上看，它内部有一个抽象 类 Sync，是继承了AQS，自己实现的一个同步器。同时，ReentrantLock内部有两个非抽象类 NonfairSync 和 FairSync ，它们都继承了Sync。从名字上看得出，分别是”非公平同步器“和”公平同步器“的意思。这意味着ReentrantLock可以支持”公平锁“和”非公平锁“。

通过看着两个同步器的源码可以发现，它们的实现都是”独占“的。都调用了AOS的 setExclusiveOwnerThread方法，所以ReentrantLock的锁的”独占“的，也就是说，它的锁都是”排他锁“，不能共享。

在ReentrantLock的构造方法里，可以传入一个 boolean 类型的参数，来指定它是否是一个公平锁，默认情况下是非公平的。这个参数一旦实例化后就不能修改，只能通过 isFair() 方法来查看。

**ReentrantReadWriteLock**

这个类也是一个非抽象类，它是ReadWriteLock接口的JDK默认实现。它与ReentrantLock的功能类似，同样是可重入的，支持非公平锁和公平锁。不同的是，它还支持”读写锁“。

```java
// 内部结构
private final ReentrantReadWriteLock.ReadLock readerLock;
private final ReentrantReadWriteLock.WriteLock writerLock;
final Sync sync;
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 具体实现 
}
static final class NonfairSync extends Sync {
    // 具体实现 
}
static final class FairSync extends Sync {
    // 具体实现 
}
public static class ReadLock implements Lock, java.io.Serializable {
   private final Sync sync;
   protected ReadLock(ReentrantReadWriteLock lock) {
   sync = lock.sync;
   }
   // 具体实现 
}
public static class WriteLock implements Lock, java.io.Serializable {
   private final Sync sync;
   protected WriteLock(ReentrantReadWriteLock lock) {
   sync = lock.sync;
   }
   // 具体实现 
}
// 构造方法，初始化两个锁
public ReentrantReadWriteLock(boolean fair) {
   sync = fair ? new FairSync() : new NonfairSync();
   readerLock = new ReadLock(this);
   writerLock = new WriteLock(this);
}
// 获取读锁和写锁的方法
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock readLock() { return readerLock; }
```

ReentrantReadWriteLock实现了读写锁，但它有一个弊端，就是在“写”操作的时候，其它线程不能写也不能读。我们称这种现象为“写饥饿”，将在后面的StampedLock类继续讨论这个问题。

**StampedLock**

StampedLock 类是在Java 8 才发布的，也是Doug Lea大神所写，有人号称它为锁的性能之王。它没有实现Lock接口和ReadWriteLock接口，但它其实是实现了“读写锁”的功能，并且性能比ReentrantReadWriteLock更高。StampedLock还把读锁分
为了“乐观读锁”和“悲观读锁”两种。

前面提到了ReentrantReadWriteLock会发生“写饥饿”的现象，但StampedLock不会。它是怎么做到的呢？它的核心思想在于，**在读的时候如果发生了写，应该通过重试的方式来获取新的值，而不应该阻塞写操作。这种模式也就是典型的无锁编程思想，和CAS自旋的思想一样**。这种操作方式决定了StampedLock在读线程非常多而写线程非常少的场景下非常适用，同时还避免了写饥饿情况的发生。