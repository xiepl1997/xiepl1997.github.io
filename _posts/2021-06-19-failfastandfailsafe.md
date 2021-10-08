---
layout: post
title: 【小记】fail-fast和fail-safe
date: 2021-06-19
author: xiepl1997
tags: java 小记
---

以前也遇到过在遍历集合的过程中对集合元素进行删除的时候会报出错误的情况，之前一直没弄明白是怎么回事，这次也花了一点时间看了一下相关内容，在此做一个总结。

## fail-fast
快速失败其实是一种编程思想，即**快速反馈系统错误**，防止发生更严重的问题。我们平时写的在函数的开始进行参数的判空操作，其实也是属于一种快速失败机制的实现。  

在用迭代器遍历一个对象的时候，如果遍历的过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出**Concurrent Modification Exception**异常。如下面这段代码。  
```java
public static void main(String[] args) {
	List<String> list = new ArrayList<>();
	list.add("1");
	list.add("2");
	list.add("3");
	for (String s : list) {
		System.out.println(s);
		list.remove(s);
	}
	System.out.println(list.size());
}
```
那是什么原因导致的异常呢？  

## fail-fast原理
上面的代码通过反编译，发现底层使用的是迭代器进行集合遍历。
```java
public static void main(String[] args) {
	List<String> list = new ArrayList<>();
	list.add("1");
	list.add("2");
	list.add("3");
	Iterator var2 = list.iterator();

	while (var2.hasNext()) {
		String s = (String)var2.next();
		System.out.println(s);
		list.remove(s);
	}
	System.out.println(list.size());
}
```
再看迭代器遍历元素时使用的next()方法。
```java
private class Itr implements Iterator<E> {
	int cursor;
 	int lastRet = -1; 
 	int expectedModCount = modCount;
	Itr() {}

	public boolean hasNext() {
		return cursor != size;
	}

	@SuppressWarnings("unchecked")
	public E next() {
		checkForComodification();
 		int i = cursor;
		if (i >= size)
			throw new NoSuchElementException();
		Object[] elementData = ArrayList.this.elementData;
		if (i >= elementData.length)
			throw new ConcurrentModificationException();
		cursor = i + 1;
		return (E) elementData[lastRet = i];
	}
}
```
在遍历前需要检查，看到checkForComodification方法，这里可能会抛出一个ConcurrentModificationException异常，依据的是modCount != expectedModCount条件。
```java
final void checkForComodification() {
	if (modCount != expectedModCount)
 		throw new ConcurrentModificationException();
}
```
这里涉及到两个变量
* modCount：记录集合的修改次数，任何涉及元素变化的操作都会修改这个值，比如add、remove
* expectedModCount：在迭代器创建时会用expectedModCount记录那个时刻的modCount值，即expectedModCount = modCount.
expectedModCount的值是静态的，而modCount是动态变化的，这就很好理解上面代码为什么会抛出ConcurrentModificationException异常了，在遍历集合的过程中，同时修改了集合元素导致modCount变化了，使得expectedModCount != modCount

## 设计思想
上述例子，通过单线程模拟了ArrayList存在的线程安全问题，即多线程并发的情况下，可能存在A线程正在遍历集合，B线程同时修改了集合元素的问题，另外通过fail-fast机制将安全隐患通知到开发者。  
ArrayList是非同步容器，fail-fast是为可能发生的并发问题提供一种预警机制，这也是ConcurrentModificationException异常名称由来。  
java.util包下的集合类都是有快速失败机制的，不能在多线程环境下发生并发修改（迭代过程中被修改）。

***

## fail-safe
讲安全失败前看一个例子，这个例子和快速失败的例子差不多，只不过替换了容器。
```java
public static void main(String[] args) {
	CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
	list.add("a");
	list.add("b");
	list.add("c");
	for (String s : list) {
		System.out.println(s);
		list.remove(s);
	}
	System.out.println(list.size());
}
```
发现上述代码能够正常运行，没有发生任何异常。这就是fail-safe安全失败机制，而CopyOnWriteArrayList正是实现了这种机制来预防线程安全问题。

## 与fail-fast的区别
在jdk中其实没有fail-safe的定义，本质上是用来区分fail-fast而衍生出的一种说法，它与fail-fast区别在于：
* fail-safe迭代器允许在遍历集合的同时，修改集合的结构，且不会抛出异常。
* fail-safe迭代器会创建原始集合的副本，在新副本的基础上进行修改操作，因此需要额外申请内存空间。
fail-safe最经典的实现便是COW（Copy On Wrote），即写时复制，另外COW只有在进行修改操作时才会发生集合的复制。

## CopyOnWriteArrayList实现原理
首先看CopyOnWriteArrayList内部迭代器的主要代码
```java
static final class COWIterator<E> implemets ListIterator<E> {
	// 数组的快照
	private final Object[] snapshot;
	private int cursor;

	private boolean hasNext() {
		return cursor < snapshot.length;
	}
	public boolean hasPrevious() {
		return cursor > 0;
	}

	@SuppressWarnings("unchecked")
	public E next() {
	if (! hasNext())
		throw new NoSuchElementException();
		return (E) snapshot[cursor++];
	}
}
```
代码非常简洁，与ArrayList迭代器不同的是：
* 在创建迭代器的同时，其中的成员变量snapshot会指向当前集合的数组对象
* 在使用next()方法遍历集合的时候，并没有checkForComodification的检查
啥检查没有，CopyOnWriteArrayList怎么解决线程安全的问题？接下来看remove()方法。
```java
private boolean remove(Object o, Object[] snapshot, int index) {
	final ReentrantLock lock = this.lock;
	lock.lock();
	try {
 		Object[] current = getArray();
		int len = current.length;
		// 省略索引检查相关的代码，不是本文讨论的重点
		Object[] newElements = new Object[len - 1];
		// 创建了一个当前集合数组的副本 newElements
		System.arraycopy(current, 0, newElements, 0, index);
		// 在副本 newElements 的基础上对元素进行修改操作
		System.arraycopy(current, index + 1,
							newElements, index,
							len - index - 1);
		// 最后用修改后的副本 newElements 覆盖原始集合数组
		setArray(newElements);
 		return true;
	} finally {
	lock.unlock();
	}
}
```
步骤如下：
* 创建了一个当前集合数组的副本newElements
* 在副本newElements的基础上对元素进行修改操作
* 最后用修改后的副本newElements覆盖原始集合数组（改变引用指向）
现在应该明白了迭代器中的那个snapshot变量命名的意义了，因为它一旦被创建，就不会再改变了，可以认为是那个时刻集合元素的快照，**任何涉及到集合元素改变的操作，都是在新的集合副本中进行，最后再让集合数组引用指向新的集合副本**。  

但需要注意，在拷贝副本时还是进行了加锁操作，防止多个线程同时创建多份副本，如果那样还是线程不安全；另外写时复制只能保证数据的最终一致性。  

java.util.concurrent包下的容器都是fail-safe的，可以在多线程下并发使用，并发修改。