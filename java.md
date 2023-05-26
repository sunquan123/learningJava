# Java面试题

## 父子类的构造顺序

父类静态代码块->子类静态代码块->父类非静态代码块->父类构造函数->子类非静态代码块->子类构造函数

## CopyOnWriteArrayList和ReadWriteLock场景

CopyOnWriteArrayList适用于写少读多的并发场景

ReadWriteLock即为读写锁，他要求写与写之间互斥，读与写之间互斥，    读与读之间可以并发执行。在读多写少的情况下可以提高效率

## switch语句

switch语句后的控制表达式只能是short、char、int、long整数类型和枚举类型，不能是float，double和boolean类型。String类型是java7开始支持。

## ThreadLocal类

ThreadLocal是采用哈希表的方式来为每个线程都提供一个变量的副本

ThreadLocal保证各个线程间数据安全，每个线程的数据不会被另外线程访问和破坏

## 哪种情况会导致持久区jvm堆内存溢出？

Java中堆内存分为两部分，分别是permantspace和heap space。permantspace（持久区）主要存放的是Java类定义信息，与垃圾收集器要收集的Java对象关系不大。持久代溢出通常由于持久代设置过小，动态加载了大量Java类，因此C选项正确。

heap space分为年轻代和年老代， 年老代常见的内存溢出原因有循环上万次的字符串处理、在一段代码内申请上百M甚至上G的内存和创建成千上万的对象。

## Java异常体系

![6479D7BB01736CCC61B8270D41F00B17.png](C:\Users\admin、\Desktop\6479D7BB01736CCC61B8270D41F00B17.png)

## 默认RMI采用的是什么通信协议？

RMI (Remote Method Invocation) 默认采用的是 **IIOP** (Internet Inter-ORB Protocol) 通信协议。 IIOP 是一种基于 TCP/IP 的通信协议，它是由 CORBA (Common Object Request Broker Architecture) 组织定义的。 RMI 通过 IIOP 协议在不同的 Java 虚拟机之间进行远程方法调用。
