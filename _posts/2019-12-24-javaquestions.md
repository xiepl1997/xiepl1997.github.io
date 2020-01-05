---
layout: post
title: Java专业术语五十问(上)
date: 2019-12-25
author: xiepl1997
tags: java
---

<font color="#FF0000">StringBuffer和StringBuilder区别是什么？</font>  
String是不可变字符序列  
StringBuffer可变字符序列，线程安全  
StringBuilder可变字符序列，线程不安全  

<font color="#FF0000">线程安全是什么？</font>  
线程安全就是多线程访问时，采用了加锁机制，当一个线程访问该类的某个数据时，进行保护，其他线程不能进行访问直到该先线程读取完，其他线程才可以使用。不会出现数据不一致或者数据污染。  
线程不安全就是不提供数据访问保护，有可能出现多个线程先后更改数据造成所得到的数据是脏数据  

<font color="#FF0000">什么是死锁？</font>  
所谓死锁，是指多个进程在运行过程中因争夺资源而造成的一种僵局，当进程处于这种僵持状态时，若无外力作用，它们都将无法再向前推进。  
原因为两点：竞争资源、进程间推进顺序非法。  
死锁产生的四个必要条件：互斥条件、请求和保持条件、不剥夺条件、环路等待条件。  

<font color="#FF0000">synchronized的实现原理是什么？</font>  
待更  

<font color="#FF0000">有了synchronized，还需要volatile做什么事？</font>  
待更  

<font color="#FF0000">synchronized的锁优化是怎么处理的？</font>  
待更  

<font color="#FF0000">JMM是什么？</font>  
java memory model，java内存模型。JMM主要规定了线程和内存之间的关系。根据JMM的设计，系统存在一个主内存，java中所有的实例变量都存储在主存中，对于所有的线程来说是共享的。在java多线程场景下，系统会为每一个线程开辟一个独立的本地内存，也叫工作内存，线程运行时，会将主内存中的数据拷贝到各自的工作内存中，每个工作内存之间对数据的修改是不可见的，使用volatile关键字可以使线程之间的数据修改变得可见。  
JMM屏蔽了各种硬件和操作系统中的内存访问差异，以实现让java程序在各种平台下都能达到一致的内存访问效果。  

<font color="#FF0000">Java并发包都有哪些？</font>  
locks部分：包含在java.util.concurrent.locks包中，提供显示锁（互斥锁和速写锁）相关功能。  
atomic部分：包含在java.util.concurrent.atomic包中，提供原子变量类相关的功能，是构建非阻塞算法的基础。  
executor部分：散落在java.util.concurrent包中，提供线程池相关的功能。  
collections部分：java.util.concurrent包中，提供并发容器相关功能。  
tools部分：散落在java.util.concurrent包中，提供同步工具类，如信号量、闭锁、栅栏等功能。  

<font color="#FF0000">什么是fail-fast？</font>  
fail-fast机制是java集合（Collections）中的一种错误机制。当多个线程对同一个集合中的内容进行操作时，就可能会产生fail-fast事件。  
例如，当某一个线程A通过iterator去遍历某集合中的过程中，若该集合的结构被其他线程所改变了，那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。  

<font color="#FF0000">什么是fail-safe？</font>  
当对集合结构上做出改变时，fail-fast机制会抛出异常，但是，对于采用fail-safe机制来说，就不会抛出异常，这是因为，当集合结构被改变时，fail-safe机制会再复制原集合的一份数据出来，然后在复制出来的那份数据上进行遍历。  

<font color="#FF0000">什么是CopyOnWrite？</font>  
Java中提供两个的CopyOnWrite容器，一个是CopyOnWriteArrayList和CopyOnWriteArraySet。  
CopyOnWrite容器即写时复制容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是将当前容器进行Copy，复制出一个新容器，然后往新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。  
CopyOnWrite并发容器用于读多写少的并发场景。  

<font color="#FF0000">什么是AQS？</font>  
待更  

<font color="#FF0000">什么是CAS？</font>  
待  

<font color="#FF0000">乐观锁是什么？</font>  
乐观锁是相对悲观锁而言的，乐观锁假设数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则返回给用户错误的信息，让用户决定如何去做。  
<font color="#FF0000">乐观锁和悲观锁的区别是什么？</font>  
<font color="#FF0000">数据库如何实现悲观锁和乐观锁？</font>  
<font color="#FF0000">数据库锁和隔离级别有什么关系？</font>  
<font color="#FF0000">数据库锁和索引有什么关系？</font>  
<font color="#FF0000">什么是聚簇索引？</font>  
聚簇索引也叫簇类索引，是一种对磁盘上实际数据重新组织以按指定的一个或多个列的值排序。由于聚簇索引的索引页面指针指向数据页面，所以使用聚簇索引查找数据几乎总是比使用非聚簇索引快。每张表只能建一个聚簇索引，并且建聚簇索引需要至少相当该表120%的附加空间，以存放该表的副本和索引中间页。  
<font color="#FF0000">什么是非聚簇索引？</font>  
<font color="#FF0000">索引最左前缀是什么？</font>  
<font color="#FF0000">什么是B+数索引？</font>  
<font color="#FF0000">什么是联合索引？</font>  
<font color="#FF0000">是什么回表？</font>  
<font color="#FF0000">分布式锁有了解吗？</font>  
