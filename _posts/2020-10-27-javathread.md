---
layout: post
title: Java多线程的创建
date: 2020-10-27
author: xiepl1997
tags: java
---

做实验的过程中需要用到多线程，许久不用有点儿生疏了，现在查资料做个多线程创建的方法记录。  

Java使用线程大致有以下四种方法：  
* 继承Thread类，重写run方法。（Thread类本身也实现了Runnable接口）
* 实现Runnable接口，重写run方法。
* 实现Callable接口，重写call方法（有返回值）
* 使用线程池（有返回值）

## 1.继承Thread类，重写run方法
每次创建一个新的线程，都需要新建一个Thread子类的对象。  
一个线程调用两次start()方法将会抛出线程状态异常。  
**继承Thread类的方式是多个线程各自完成各自的任务。**  
创建线程实际调用的是父类Thread空参的构造器。
```java
public class Test {
	public static void main(String[] args) {
		for (int i = 0; i < 10; i++) {
			MyThread thread = new MyThread();
			thread.satrt();
		}
		System.out.println("This is main thread");
	}
}

public class MyThread extends Thread {
	@Override
	public void run() {
		System.out.println("This is MyThread.");
	}
}
```

## 2.实现Runnable接口，重写run()方法
覆写Runnable接口实现多线程，可以避免单继承局限。不论创建多少个线程，只需要创建一个Runnable接口实现类的对象。  
创建线程实际调用的Thread类Runnable类型参数的构造器。  
**实现Runnable接口的方法可以多个线程共同完成一个任务。**
```java
public class Test {
	public static void main(String[] args) {
		Task task = new Task();
		for (int i = 0; i < 10; i++) {
			Thread thread = new Thread(task);
			thread.satrt();
		}
		System.out.println("This is main thread");
	}
}

public class Task implements Runnable {
	@Override
	public void run() {
		System.out.println("This is MyThread.");
	}
}
```

## 3.实现Callable接口，重写call()方法
自定义类实现Callable接口时，必须指定泛型，该泛型即返回值的类型。  
每次创建一个新的线程，都需要创建一个新的Callable接口的实现类。  
启动线程的过程：
* 创建一个Callable接口的实现类的对象实例。
* 创建一个FutureTask对象，传入Callable类型的参数。
* 调用Thread类重载参数为Runnable的构造器创建Thread对象，将FutureTask作为参数传入。
线程允许完后获取返回值：
* 使用FutureTask的get()方法。  

```java
public class Test {
	public static void main(String[] args) {
		for (int i = 0; i < 10; i++) {
			Task task = new Task();
			FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
            Thread thread = new Thread(futureTask);
            thread.start();
            try {
                System.out.println("The result is : " + futureTask.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
		}
		System.out.println("This is main thread");
	}
}

public class Task implements Callable<Integer> {
	@Override
	public Integer call() {
		int result = 0;
		for (int i = 0; i < 10000; i++)
			result += i;
		return result;
	}
}
```
FutureTask的get()方法在调用的时候会阻塞，直到当前的线程完成运行后将结果返回。

## 4.使用线程池
在java.util.concurrent包下，提供了一系列与线程池有关的类。合理使用线程池，可以带来诸多好处：
* **降低资源消耗**，通过重复利用已创建的线程来降低线程创建和销毁造成的消耗。
* **提高响应速度**，当任务到达时，任务可以不必等到新线程创建就能立即运行。
* **提高线程的可管理性**，线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配、调优和监控。
线程池可以应对突然大爆发的访问，通过有限个固定线程为大量的操作服务，减少创建和销毁线程所需的时间。
使用线程池的过程：  
* 创建线程池。
* 创建任务。
* 执行任务。
* 关闭线程池。

这里使用一个固定大小的线程池来举例子。  
```java
public class Test {

	private static final int THREAD_POOL_SIZE = 5;

	public static void main(String[] args) {
		ExecutorService executorService = Executors.newFixedThreadPool(THREAD_POOL_SIZE);
		Task task = new Task();
		FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
		executorService.submit(futureTask);
		executorService.shutdown();
		try {
			System.out.println("The result is : " + futureTask.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
		System.out.println("This is main thread");
	}
}

public class Task implements Callable<Integer> {
	@Override
	public Integer call() {
		int result = 0;
		for (int i = 0; i < 10000; i++)
			result += i;
		return result;
	}
}
```

## 5.对比
可以将java中的线程创建方式分为两大类：一类是继承Thread类实现多线程；另一类是通过实现Runnable或者Callable接口实现多线程。  
下面来分析一下这两类实现多线程方式的优劣：  

### 通过继承Thread实现多线程
* 优点  
1.实现简单，而且要获取当前线程，无需调用Thread.currentThread()方法，直接使用this即可获取当前线程。  
* 缺点  
1.线程已经继承了Thread类了，就不能继承其他类。  
2.多个线程不能共享同一份资源。  

### 通过实现Runnable或Callable接口实现多线程
* 优点  
1.线程类只是实现了接口，还可以继承其他类。  
2.多个线程可以使用同一个target对象，适合多个线程处理同一个任务的情况。  
* 缺点  
1.较第一类方法，编程较为复杂。  
2.要访问当前线程，必须使用Thread.currentThread()方法。

### 综上 ：使用第二类方法较多！