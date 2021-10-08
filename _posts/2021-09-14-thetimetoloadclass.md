---
layout: post
title: 【小记】探析Java类加载时机
date: 2021-09-14
author: xiepl1997
tags: java jvm 小记
---

最近在重温《深入理解Java虚拟机》这本书，对于第七章的类加载机制部分了解到了之前没有注意的细节，特在此总结记录下来。  

---

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）7个阶段。其中验证、准备、解析3个阶段部分统称为连接（Linking）。  

什么情况下需要开始类加载过程的第一个阶段：加载？Java虚拟机规范中并没有进行强制约束，这点可以交给虚拟机的具体实现来自由把握。但是对于初始化阶段，虚拟机规范则是严格规定了**有且只有5种**情况必须立即对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）  

### 五种情况
* 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
* 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
* 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
* 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机回先初始化这个主类。
* 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。  

对于这5种会触发类进行初始化的场景，虚拟机规范中使用了一个很强烈的限定语：**“有且只有”**，这5种场景种的行为称为对一个类进行**主动引用**。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。下面举3个例子来说明何为**被动引用**。

#### 被动引用的例子1
```java
package org.fenixsoft.classloading

/**
 * 被动使用类字段例1；
 * 通过子类引用父类的静态字段，不会导致子类初始化
 **/
public class SuperClass {
	static {
		System.out.println("SuperClass init!");
	}
	public static int value = 123;
}

public class SubClass extends SuperClass {
	static {
		System.out.println("SubClass init!");
	}
}

public class NotInitialization {
	public static void main(String[] args) {
		System.out.println(SubClass.value);
	}
}
```
上述代码运行之后，只会输出“SuperClass init!”，而不会输出“SubClass init!”。对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。至于是否要触发子类的加载和验证，在虚拟机规范中并未明确规定，这点取决于虚拟机的具体实现。对于Sun HotSpot虚拟机来说，可通过-XX:+TraceClassLoading参数观察到此操作会导致子类的加载。

#### 被动引用的例子2
```java
package org.fenixsoft.classloading

/**
 * 被动使用类字段例2
 * 通过数组定义来引用类，不会触发此类的初始化
 */
public class NotInitialization {
	public static void main(String[] args) {
		SuperClass[] sca = new SuperClass[10];
	}
}
```
这段代码复用了例1，运行之后发现没有输出“SuperClass init!”，说明并没有触发类org.fenixsoft.classloading.SuperClass的初始化阶段。但是这段代码里面触发了另外一个名为Lorg.fenixsoft.classloading.SuperClass的类的初始化阶段，对于用户代码来说，这并不是一个合法的类名称，它是一个由虚拟机自动生成的、直接继承于java.lang.Object的子类，创建动作由字节码指令newarray触发。  

这个类代表了一个元素类型为org.fenixsoft.classloading.SuperClass的一维数组，数组中应有的属性和方法（用户可直接使用的只有被修饰为public的length属性和clone()方法）都实现在这个类里。Java语言中对数组的访问比C/C++相对安全是因为这个类封装了数组元素的访问方法，而C/C++直接翻译为对数组指针的移动。在Java语言中，当检查到发生数组越界时会抛出java.lang.ArrayIndexOutOfBoundException异常。

#### 被动引用的例子3
```java
package org.fenixsoft.classloading

/**
 * 被动使用类字段例3
 * 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。
 */
public class ConstClass {
	static {
		System.out.println("ConstClass init!");
	}
	public static final String HELLOWORLD = "hello world";
}
/**
 * 非主动使用类字段演示
 **/
public class NotInitialization {
	public static void main(String[] args) {
		System.out.println(ConstClass.HELLOWORLD);
	}
}
```
上述代码运行之后，也没有输出“ConstClass init!”，这是因为虽然在Java源码中引用了ConstClass类中的常量HELLOWORLD，但其实在编译阶段通过常量传播优化，已经将此常量的值“hello world”存储到了NotInitialization类的常量池中，以后NotInitialization对常量ConstClass.HELLOWORLD的引用实际都被转化为NotInitialization类对自身常量池的引用了。也就是说，实际上NotInitialization的Class文件之中并没有ConstClass类的符号引用入口，这两个类在编译成Class之后就不存在任何联系了。  

接口的加载过程与类加载过程稍有些不同。针对接口需要做一些特殊说明：接口也有初始化过程，这点与类是一致的，上面的代码都是用静态语句块“static{}”来输出初始化信息的，而接口中不能使用“static{}”语句块，但编译器仍然会为接口生成“<clinit>()”类构造器，用于初始化接口中所定义的成员变量。接口与类真正有所区别的是前面讲述的5中“有且仅有”需要开始初始化的场景中的第3种：当一个类在初始化时，要求其父类全部都已经初始化过了，但是一个接口在初始化时，并不要求其父接口全部都完成了初始化，只是在真正使用到父接口的时候（如引用接口中定义的常量）才会初始化。