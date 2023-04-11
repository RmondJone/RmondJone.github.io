---
title: "《深入浅出Java多线程》--基础篇"
date: 2023-04-11T11:22:19+08:00
draft: false
categories: ["Java"]
tags: ["Java"]
---

## 一、进程与线程的基本概念
**进程**：就是应用程序在内存中分配的空间，也就是正在运行的程序，各个进程之间互不干扰。同时进程保存着程序每一个时刻运行的状态。（例如手机里的一个个App,一个App的运行就对应一个主进程）
**线程**：如果一个进程有多个子任务时，只能逐 个得执行这些子任务，很影响效率。于是人们提出了线程的概念，让一个线程 执多个子任务，这样一个进程就包含了多个线程，每个线程负责一个单独的子任务。（例如迅雷App,一个下载任务就对应着一个线程）

总之，进程和线程的提出极大的提高了操作提供的性能。进程让操作系统的并发性成为了可能，而线程让进程的内部并发成为了可能。

### 进程和线程的区别

进程是一个独立的运行环境，而线程是在进程中执行的一个任务。他们两个本质的区别是**是否单独占有内存地址空间及其它系统资源（例如I/O）**：
* 进程单独占有一定的内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂但是同步简单，各个进程之间互不干扰；而线程共享所属进程占有的内存地址空间和资源，数据共享简单，但是同步复杂。
* 进程单独占有一定的内存地址空间，一个进程出现问题不会影响其他进程，不影响主程序的稳定性，可靠性高；一个线程崩溃可能影响整个程序的稳定性， 可靠性较低。
* 进程单独占有一定的内存地址空间，进程的创建和销毁不仅需要保存寄存器和栈信息，还需要资源的分配回收以及页调度，开销较大；线程只需要保存寄存器和栈信息，开销较小。

另外一个重要区别是，进程是**操作系统进行资源分配的基本单位，而线程是操作系统进行调度的基本单位**，即CPU分配时间的单位 。

## 二、Java多线程入门类和接口
JDK提供了 Thread 类和 Runnalble 接口来让我们实现自己的“线程”类：
* 继承Thread 类并重写run()
* 实现Runnalble 接口的run()
```java
public class Demo {
  public static class MyThread extends Thread {
    @Override
    public void run() {
      System.out.println("MyThread");
    }
  }
  public static void main(String[] args) {
    Thread myThread = new MyThread();
     myThread.start();
  }
}
```
```java
public class Demo {
   public static class MyThread implements Runnable {
     @Override
     public void run() {
        System.out.println("MyThread");
     }
   }
   public static void main(String[] args) {
     new MyThread().start();
     // Java 8 函数式编程，可以省略MyThread类
     new Thread(() -> {
        System.out.println("Java 8 匿名内部类");
     }).start();
   }
}
```
Thread类与Runnable接口的比较：
* 由于Java“单继承，多实现”的特性，Runnable接口使用起来比Thread更灵活。
* Runnable接口出现更符合面向对象，将线程单独进行对象的封装。
* Runnable接口出现，降低了线程对象和线程任务的耦合性。

## 三、Java线程的状态及主要转化方法
Java线程的6个状态如下：
```java
public enum State {
 NEW,//新建
 RUNNABLE,//运行
 BLOCKED,//锁定
 WAITING,//等待
 TIMED_WAITING,//定时等待
 TERMINATED;//终止
}
```
![线程状态的转换](/images/java_base_thread_1.webp)
## 四、线程之间的通信
线程的通信是指线程之间以何种机制来交换信息。在编程中，线程之间的通信机制有两种，**共享内存**和**消息传递**。

（1）在**共享内存的并发模型**里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信，典型的共享内存通信方式就是通过共享对象进行通信。

![共享内存的并发模型](/images/java_base_thread_1.webp)

采用这种的通讯方式的涉及的知识点有：
* 锁（synchronized）
* 信号量（volatile、Semaphore）

（2）在**消息传递的并发模型**里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。

采用这种的通讯方式的涉及的知识点有：
* 基于 Object 类的 wait() 方法和 notify()通知/等待机制实现的线程通讯方式。
* 基于“管道流”的通信方式（PipedWriter 、 PipedReader 、
  PipedOutputStream 、 PipedInputStream 。其中，前面两个是基于字符的，后面两个是基于字节流的）实现的线程通讯方式。



