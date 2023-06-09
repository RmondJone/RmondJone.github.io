---
title: "Java抽象类与接口的区别"
date: 2023-04-10T17:40:36+08:00
draft: false
categories: ["Java"]
tags: ["Java"]
---

## 抽象类和接口的对比

参数|抽象类|接口
--|--|--
默认的方法实现	|它可以有默认的方法实现	|接口完全是抽象的。它根本不存在方法的实现
实现	|子类使用extends关键字来继承抽象类。子类不用实现所有声明的方法，但如果是被abstract关键字标记的方法则必须实现|	子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现
构造器	|抽象类可以有构造器	|接口不能有构造器
与正常Java类的区别	|除了你不能实例化抽象类之外，它和普通Java类没有任何区别	|接口是完全不同的类型
访问修饰符|	抽象方法可以有public、protected和default这些修饰符	|接口方法默认修饰符是public。你不可以使用其它修饰符。
main方法	|抽象方法可以有main方法并且我们可以运行它	|接口没有main方法，因此我们不能运行它。
多继承	|抽象方法可以继承一个类和实现多个接口	|接口只可以继承一个或多个其它接口
速度	|它比接口速度要快	|接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。
添加新方法	|如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。	|如果你往接口中添加方法，那么你必须改变实现该接口的类。

## 什么时候使用抽象类和接口？

* 如果你拥有一些方法并且想让它们中的一些有默认实现，那么使用抽象类吧。
* 如果你想实现多重继承，那么你必须使用接口。由于Java不支持多继承，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。
* 如果基本功能在不断改变，那么就需要使用抽象类。如果不断改变基本功能并且使用接口，那么就需要改变所有实现了该接口的类。