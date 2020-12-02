---
layout: post
title: Comparator为何是函数式接口？
date: 2020-12-01
author: xiepl1997
tags: java
---

## Functional Interface
Java SE 8中重磅推出了lambda表达式，为了实现lambda进而又新增了函数式接口：对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式，这种接口称为**函数式接口（functional interface）**  

Java的函数式接口与函数式编程有着重要的联系。函数式编程最大的特点就是**函数（function）作为一等公民**。所谓一等公民，说明了它在编程语言中的重要性。我们的传统语言中大部分是将**值**作为一等公民，而函数方法大多以API的方式去处理和传递值，类和结构体就像一个包裹，他们负责如何去组合值。函数式编程让函数和值收到了平等对待。  

Java最开始也是将值作为一等公民，但随着函数式编程的流行以及它带来的方便受人喜爱，Java在JDK1.8版本中推出了函数式编程的解决方法，**函数式接口**。Java作为一门高级语言，有着极高的安全性，但没有C/C++这种底层语言所具有的灵活性，在C/C++中可以传递一个函数的引用地址，进而将函数地位提升，升级到一等公民的地位。但是在Java中没有指针，所以Java中传递函数就变得困难。但是Java可以通过函数式接口的方式结合lambda表达式实现函数式编程。

## Comparator为何是函数式接口？
介绍完了函数式接口，问题回来了，为何Comparator是函数式接口呢？来看Comparator的源码。
```java
@FunctionalInterface
public interface Comparator<T> {
	
	int compare(T o1, T o2);

	boolean equals(Object obj);

	default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    public static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(true, comparator);
    }

    ....
    ....
    ....
}
```
可以看到，明明有好多方法呀！？再细看，equals是Object中的方法，这种对Object类的方法的重新声明会让方法不再是抽象的。（Java API中的一些接口会重新声明Object方法来附加javadoc注释。）更重要的是，JDK1.8中，接口可以声明非抽象方法了，例如上面代码中的default方法和static方法，且方法中可以有具体实现！！！这是JDK1.8的一个重要的变化。。记住。最后就只有一个抽象方法compare了，所以Comparator是符合函数式接口规范的。
