---
layout: post
title: HashMap源码阅读
date: 2020-02-02
author: xiepl1997
tags: 源码阅读
---

下面是JDK11中HashMap的源码分析，对代码的分析将主要以注释的方式来体现。  

## 1 概述

### 1.1 HashMap的主要概念  
1. HashMap是基于Map接口实现的哈希表，实现了Map接口中的所有操作，而且HashMap允许键为空值，也允许值为空值，与之对应的是Hashtable，Hashtable不能将键和值设置为空。HashMap不能保证元素的顺序，特别是，它不能保证随着时间的推移保持顺序不变。  

2. HashMap为基本操作（get和put）提供了恒定的时间性能，假设散列函数在木桶（buckets）中适当地分散了元素。集合（Collection）的迭代需要的时间与HashMap实例的“容量”和它的大小成比例。因此，如果迭代的性能很重要，要求很高，那么不将初始容量设置得太高（或负载因素过低）是非常重要的。  

3. HashMap是线程不安全的集合，即当多线程访问时，同一时刻如果无法保证只有一个线程修改HashMap，则会毁坏HashMap，抛出ConcurrentModificationException。  

### 1.2 HashMap的基本实现  
HashMap底层使用哈希表（数组 + 单链表），当链表过长会将链表转成红黑树以实现O(logn)时间复杂度内查找。  
HashMap的定义为```class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable```  

### 1.3 HashMap内部类
1.Node  
2.KeySet  
3.Values  
4.EntrySet  
5.HashIterator  
6.KeyIterator  
7.ValueIterator  
8.EntryIterator  
9.HashMapSpliterator  
10.KeySpliterator  
11.ValueSpliterator  
12.EntrySpliterator
13.TreeNode  代表红黑树节点，HashMap中对红黑树的操作的方法都在此类中

### 1.4 扩容原理
HashMap采用的扩容策略是，每次加倍的方式。这样，原来位置的Entry在新扩展的数组中要么依然在原来的位置，要么在```原来的位置+原来的容量```的位置。

### 1.5 hash计算
HashMap通过hash()函数（也叫“扰动函数”）来计算hash值，方法为```key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;```，计算出来的hash值存放在Node.hash中。  

hash的值计算相当于将高16位与底16位进行异或，结果是高16位不变，底16位变成异或的新结果。为什么这样做呢，原因是HashMap扩容之前的数组大小才为16，散列值是不能直接拿来用的。在进行长度取模运算时采用的只是取二进制中的最右端的几位，并没有用到高位二进制的信息，做带来的结果就是hash结果分布不太均匀。而将高16位和底16位异或后就可以让低位附带高位的信息，加大低位的随机性。  

在对散列值做完高低位的异或操作后，在对异或结果进行对长度的取模得到最终的结果。具体参考[JDK源码中HashMap的hash方法原理是什么？-胖君的回答-知乎](https://www.zhihu.com/question/20733617/answer/111577937)

### 1.6 插入null原理
在hash计算中，null的hash值为0，然后按照正常的```putVal()```插入。

### 1.7 new HashMap()
从源码中（下文构造函数）我们可以看到，new HashMap()开销非常少，仅仅确认装载因子。<u>真正的创建table的操作尽可能的往后延迟</u>，这使得HashMap有不少操作都需要检查table是否初始化。这种设计有一种好处，就是能够不必担心HashMap的开销，可以一次性大量的创建HashMap。

## 2 HashMap的成员变量
```java
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
	//用于序列化
	private static final long serialVersionUID = 362498820763181265L;
	//HashMap的默认容量是16
	static final int DEFAULT_INITIAL_CAPACITY = 16;
	//最大容量为1073741824（2的30次方，即1<<30）
	static final int MAXIMUM_CAPACITY = 1073741824;
	//默认装载因子为0.75f
	static final float DEFAULT_LOAD_FACTOR = 0.75F;
	/*
	将链表转化为红黑树的阈值为8，即当链表长度 >= 8时，链表转化为红黑树，也就是树形化。
	为什么要树形化呢？想一下我们为什么要用HashMap，是因为通过Hash算法在理想情况下时间复杂度O(1)就能找到元素，特别快，但仅限于理想情况下，如果遇到了hash碰撞，且碰撞比较频繁的话，那么当我们get一个元素的时候，定位到了这个数组，还需要在这个数组中遍历一次链表最终才能找到要get的元素，是不是已经失去了hashmap的初心了？（因为需要遍历链表，所以时间复杂度就高上去了）。
	所以使用红黑树这种数据结构来解决链表过长的问题，可以理解为红黑树遍历比链表遍历快，时间复杂度低。
	*/
	static final int TREEIFY_THRESHOLD = 8;
	//将红黑树转化成链表的阈值为6（<6时），这个是在resize()的过程中调用TreeNode.split()实现
	static final int UNTREEIFY_THRESHOLD = 6;
	/*
	最小树形化阈值。要树化并不仅仅是要超过TREEIFY_THRESHOLD，同时容量要超过MIN_TREEIFY_CAPACITY，如果只是超过TREEIFY_THRESHOLD，则会进行扩容（调用resize()）。为什么这个时候是扩容而不是树形化呢？
	原因就在于，造成链表过长也可能是数组（桶）太短了也就是容量太小了。举个例子，如果数组长度为1，那么所有的元素都挤在了数组的第0个位置上，这个时候就算树形化只是治标不治本，因为引起链表过长的根本原因是数组过短。
	所以在执行树形化之前（链表长度>=8），会检查数组长度，如果长度小于64，则对数组进行扩容，而不是树形化。
	*/
	static final int MIN_TREEIFY_CAPACITY = 64;
	/*
	哈希表的数组主体定义，初始化时，在构造函数中并不会初始化，所以在各种操作中总是要先检查table是否为null。
	*/
	transient HashMap.Node<K, V>[] table;
	/*
	作为一个entrySet缓存，使用entrySet首先检查其是否为null，不为null则使用这个缓存，否则生成一个entrySet并缓存至此。
	*/
	transient Set<Entry<K, V>> entrySet;
	//HashMap中的Entry的数量
	transient int size;
	/*
	记录修改内部结构化修改次数，用于实现fail-fast，ConcurrentModificationException就是通过检测这个抛出。
	*/
	transient int modCount;
	//其值=capacity*loadFactor，当size超过threshold的时候便进行一次扩容
	int threshold;
	//装载因子
	final float loadFactor;

	……
}
```
## 3 HashMap的方法
### 3.1 构造函数
1. 该构造函数进行对容量和装载因子的合法性验证，然而没有对容量进行存储，只是用来确定扩容阈值threshold。
```java
public HashMap(int initialCapacity, float loadFactor) {
	if (initialCapacity < 0) {
		throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
	} else {
		if (initialCapacity > 1073741824) {
			initialCapacity = 1073741824;
		}

		if (loadFactor > 0.0F && !Float.isNaN(loadFactor)) {
			this.loadFactor = loadFactor;
			this.threshold = tableSizeFor(initialCapacity);
		} else {
			throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
		}
	}
}
```
2. 只传入初始容量
```java
public HashMap(int initialCapacity) {
	this(initialCapacity, 0.75F);
}
```
3. 无参构造函数仅仅确认装载因子
```java
public HashMap() {
	this.loadFactor = 0.75F;
}
```
4. 通过Map构造HashMap时，使用默认装载因子，并调用putMapEntries将Map装入HashMap  
```java
	public HashMap(Map<? extends K, ? extends V> m) {
		this.loadFactor = 0.75F;
		this.putMapEntries(m, false);
	}

	final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
		//获取该map实际长度
	    int s = m.size();
	    if (s > 0) {
	    	//判断table是否初始化，如果没有初始化
	        if (this.table == null) {
	        	/**求出需要的容量，因为实际使用的长度=容量*0.75得来的，+1是因为小数相除，基本都不会是整数，容量大小
				不能为小数的，后面转化为int，多余的小数就要被丢掉，所以+1，例如，map实际长度为29.3，则所需要的容量为30.
	        	*/
	            float ft = (float)s / this.loadFactor + 1.0F;
	            //判断该容量大小是否超出上限
	            int t = ft < 1.07374182E9F ? (int)ft : 1073741824;
	            //对临界值进行初始化，tableSizeFor(t)这个方法会返回大于t值的，且离其最近的2次幂，例如t为29，则返回的值是32
	            if (t > this.threshold) {
	                this.threshold = tableSizeFor(t);
	            }
	        } else if (s > this.threshold) {  //如果table已经初始化，则进行扩容操作，resize就是扩容
	            this.resize();
	        }

	        Iterator var8 = m.entrySet().iterator();

	        //遍历，把map中的数据转移到hashmap中
	        while(var8.hasNext()) {
	            Entry<? extends K, ? extends V> e = (Entry)var8.next();
	            K key = e.getKey();
	            V value = e.getValue();
	            this.putVal(hash(key), key, value, false, evict);
	        }
	    }

	}
```

该构造函数，传入一个Map，然后把该Map转为hashMap，resize方法在下面添加元素的时候会详细讲解，在上面entrySet方法会返回一个Set<Map.Entry<K,V>>，泛型为Map的内部类Entry，它是一个存放key-value的实例，也就是Map中的每一个key-value就是一个Entry实例，为什么使用这个方式进行遍历，因为效率高，putVal方法把取出来的每个key-value存入到hashMap中。  

### 3.2 hash(Object key)
hash函数负责产生hashcode，计算方法为若为空则返回0，否则返回对key的高16位和底16位的异或的结果。
```java
static final int hash(Object key) {
	int h;
	return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;
}
```

### 3.3 comparableClassFor(Object x)
这个方法是判断传入的Object对象x是否实现了Comparable接口，如果传入的是String对象，自然实现了Comparable接口，直接返回就行。但是对于其他的类,比方说我们自己写了一个类对象,然后存在HashMap中,但是就HashMap来说它并不知道我们有没有实现Comparable接口,甚至都不知道我们Comparable接口中有没有用泛型,泛型具体用的是哪个类。
```java
static Class<?> comparableClassFor(Object x) {
	if (x instanceof Comparable) {
		Class c;
		if ((c = x.getClass()) == String.class) {
			return c;
		}

		ype[] ts;
		if ((ts = c.getGenericInterfaces()) != null) {
			Type[] var5 = ts;
			int var6 = ts.length;

			for(int var7 = 0; var7 < var6; ++var7) {
				Type t = var5[var7];
				Type[] as;
				ParameterizedType p;
				if (t instanceof ParameterizedType && (p = (ParameterizedType)t).getRawType() == Comparable.class && (as = p.getActualTypeArguments()) != null && as.length == 1 && as[0] == c) {
					return c;
				}
			}
		}
	}
	return null;
}
```

### 3.4 compareComparables(Class<?> kc, Object k, Object x)
如果x为空，返回0；如果x的类型为kc，则返回compareTo(x)。
```java
static int compareComparables(Class<?> kc, Object k, Object x) {
	return x != null && x.getClass() == kc ? ((Comparable)k).compareTo(x) : 0;
}
```

### 3.5 tableSizeFor(int cap)
该函数用于计算大于等于cap的的最小的2的整数幂，用于做table的长度。numberOfLeadingZeros()方法的作用是返回无符号整形i的最高非零位前面的0的个数，包括符号位在内。
```java
static final int tableSizeFor(int cap) {
	int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
	return n < 0 ? 1 : (n >= 1073741824 ? 1073741824 : n + 1);
}
```

### 3.6 put(K key, V value)
```java
	public V put(K key, V value) {
		//四个参数，第一个hash值，第四个参数表示如果该key存在值，如果为null的话，则插入新的value，最后一个参数，在hashMap中没有用，可以不用管。
        return this.putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    	//tab哈希数组，p为该哈希桶的首节点，n为hashMap的长度，i为计算出的数组下标
        HashMap.Node[] tab;
        int n;
        //获取长度并扩容，使用的是懒加载，table一开始是没有加载的，等puthou才开始加载
        if ((tab = this.table) == null || (n = tab.length) == 0) {
            n = (tab = this.resize()).length;
        }

        Object p;
        int i;
        //如果计算出的该哈希桶的位置没有值，则把新插入的key-value放到此处，此处就算没有插入成功，也就是发生哈希冲突时也会把哈希桶的首节点赋予p
        if ((p = tab[i = n - 1 & hash]) == null) {
            tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
        } else {  //发生哈希冲突的几种情况
        	//e临时节点的作用，k存放当前节点的key值
            Object e;
            Object k;
            //第一种，插入的key-value的hash值，key都与当前节点相等，e=p，则表示为首节点
            if (((HashMap.Node)p).hash == hash && ((k = ((HashMap.Node)p).key) == key || key != null && key.equals(k))) {
                e = p;
            }
            //第二种，hash值不等于首节点，判断该p是否属于红黑树的节点
            else if (p instanceof HashMap.TreeNode) {
            	/*
            	为红黑树的节点，则在红黑树中进行添加，如果该节点已经存在，则返回该节点（不为null），
            	该值很重要，用来判断put操作是否成功，如果添加成功返回null
            	*/
                e = ((HashMap.TreeNode)p).putTreeVal(this, tab, hash, key, value);
            }
            //第三种，hash值不等于首节点，不为红黑树节点，则为链表的节点
            else {
            	//遍历该链表
                int binCount = 0;
                while(true) {
                	//如果找到尾部，则表明添加的key-value没有重复，在尾部进行添加。
                    if ((e = ((HashMap.Node)p).next) == null) {
                        ((HashMap.Node)p).next = this.newNode(hash, key, value, (HashMap.Node)null);
                        //判断是否要转化为红黑树结构
                        if (binCount >= 7) {
                            this.treeifyBin(tab, hash);
                        }
                        break;
                    }

                    //如果链表有重复的key，e为当前重复的节点，结束循环。
                    if (((HashMap.Node)e).hash == hash && ((k = ((HashMap.Node)e).key) == key || key != null && key.equals(k))) {
                        break;
                    }

                    p = e;
                    ++binCount;
                }
            }
            //e不为null，则说明有重复的key，则用待插入值进行覆盖，返回旧值。
            if (e != null) {
                V oldValue = ((HashMap.Node)e).value;
                if (!onlyIfAbsent || oldValue == null) {
                    ((HashMap.Node)e).value = value;
                }

                this.afterNodeAccess((HashMap.Node)e);
                return oldValue;
            }
        }

        /*
        到了这步，说明待插入的key-value是没有key的重复，因为插入成功的e节点的值为null。
        修改次数+1
        */
        ++this.modCount;
        //实际长度+1，并判断是否大于临界值，大于则扩容
        if (++this.size > this.threshold) {
            this.resize();
        }

        this.afterNodeInsertion(evict);
        //添加成功
        return null;
    }
```

### 3.7 resize()
扩容方法resize()
```java
    final HashMap.Node<K, V>[] resize() {
    	//把没有插入之前的哈希数组叫做oldTab
        HashMap.Node<K, V>[] oldTab = this.table;
        //oldTab的长度
        int oldCap = oldTab == null ? 0 : oldTab.length;
        //oldTab的临界值
        int oldThr = this.threshold;
        //初始化new的长度和临界值
        int newThr = 0;
        int newCap;
        //oldCap>0也就说明不是首次加载，因为hashMap用的是懒加载
        if (oldCap > 0) {
        	//如果大于最大值
            if (oldCap >= 1073741824) {
            	//将临界值设置为整数的最大值
                this.threshold = 2147483647;
                return oldTab;
            }
            //位置*。其他情况，扩容两倍，并且扩容后的长度要小于最大值，old的长度也要大于16
            if ((newCap = oldCap << 1) < 1073741824 && oldCap >= 16) {
            	//临界值也要扩容为old的2倍
                newThr = oldThr << 1;
            }
        }
        /*
        如果oldCap<0，但是已经初始化了，像把元素删除完之后的情况，那么它的临界值肯定还存在，
        如果是首次初始化，它的临界值则为0.
        */
        else if (oldThr > 0) {
            newCap = oldThr;
        }
        //首次初始化，给默认值
        else {
            newCap = 16;
            newThr = 12;  //临界值等于容量*0.75
        }
        //位置*的补充，也就是初始化时容量小于默认值16的，此时newThr没有赋值
        if (newThr == 0) {
        	//new的临界值
            float ft = (float)newCap * this.loadFactor;
            //判断new容量是否大于最大值，临界值是否大于最大值
            newThr = newCap < 1073741824 && ft < 1.07374182E9F ? (int)ft : 2147483647;
        }

        //把上面各种情况分析出的临界值，在此处进行真正的改变，也就是容量和临界值都改变了
        this.threshold = newThr;
        //初始化
        HashMap.Node<K, V>[] newTab = new HashMap.Node[newCap];
        //赋予当前的table
        this.table = newTab;
        //此处自然是把old中的元素，遍历到new中
        if (oldTab != null) {
            for(int j = 0; j < oldCap; ++j) {
            	//临时变量
                HashMap.Node e;
                //当前哈希桶的位置值不为null，也就是数组下标处有值，因为有值表示可能会发生冲突
                if ((e = oldTab[j]) != null) {
                	//把已经赋值之后的变量置位null，当然是为了好回收，释放内存
                    oldTab[j] = null;
                    //如果下标处的节点没有下一个元素
                    if (e.next == null) {
                    	//把该变量的值存入newTab中，e.hash & new Cap-1并不等于j
                        newTab[e.hash & newCap - 1] = e;
                    }
                    //如果该节点为红黑树结构，也就是存在hash冲突，该hash桶中有多个元素
                    else if (e instanceof HashMap.TreeNode) {
                    	//把此树转移到newTab中
                        ((HashMap.TreeNode)e).split(this, newTab, j, oldCap);
                    }
                    /*
                    此处表示为链表结构，同样把链表转移到newTab中，就是把链表遍历后，把值转过去，再置位null
                    */
                    else {
                        HashMap.Node<K, V> loHead = null;
                        HashMap.Node<K, V> loTail = null;
                        HashMap.Node<K, V> hiHead = null;
                        HashMap.Node hiTail = null;

                        HashMap.Node next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null) {
                                    loHead = e;
                                } else {
                                    loTail.next = e;
                                }

                                loTail = e;
                            } else {
                                if (hiTail == null) {
                                    hiHead = e;
                                } else {
                                    hiTail.next = e;
                                }

                                hiTail = e;
                            }

                            e = next;
                        } while(next != null);

                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }

                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        //返回扩容后的hashmap
        return newTab;
    }
```

### 3.8 remove(Object key)
删除元素
```java
    public V remove(Object key) {
    	//临时变量
        HashMap.Node e;
        /*
        调用removeNode，第三个value表示，把key的节点直接都删除了，不需要用到值，
        如果设为值，则还需要去进行查找操作。
        */
        return (e = this.removeNode(hash(key), key, (Object)null, false, true)) == null ? null : e.value;
    }

    /*
    第一参数为哈希值，第二个为key，第三个为value，第四个为true的话，则表示删除它
    key对应的value，不删除key，第四个如果为false，则表示删除后，不移动节点。
    */
    final HashMap.Node<K, V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    	//tab哈希数组，p数组下标节点，n长度，index当前数组下标
        HashMap.Node[] tab;
        HashMap.Node p;
        int n;
        int index;
        //哈希数组不为null，且长度大于0，然后获得要删除key的节点的数组下标位置
        if ((tab = this.table) != null && (n = tab.length) > 0 && (p = tab[index = n - 1 & hash]) != null) {
        	//node存储要删除的节点，e临时变量，k当前节点的key，v当前节点的value
            HashMap.Node<K, V> node = null;
            Object k;
            //如果数组下标的节点正好是要删除的节点，把值赋给临时变量
            if (p.hash == hash && ((k = p.key) == key || key != null && key.equals(k))) {
                node = p;
            }
            //也就是要删除的节点，在链表或者红黑树上，先判断是否为红黑树的节点
            else {
                HashMap.Node e;
                if ((e = p.next) != null) {
                    if (p instanceof HashMap.TreeNode) {
                    	//遍历红黑树，找到该节点并返回
                        node = ((HashMap.TreeNode)p).getTreeNode(hash, key);
                    }
                    //如果是链表节点，遍历找到该节点
                    else {
                        label88: {
                            while(e.hash != hash || (k = e.key) != key && (key == null || !key.equals(k))) {
                            	//p为要删除节点的上一个节点
                                p = e;
                                if ((e = e.next) == null) {
                                    break label88;
                                }
                            }

                            //node为要删除的节点
                            node = e;
                        }
                    }
                }
            }

            Object v;
            /*
            找到要删除的节点后，判断!matchValue，我们正常的remove删除，!matchValue都为true
            */
            if (node != null && (!matchValue || (v = ((HashMap.Node)node).value) == value || value != null && value.equals(v))) {
                //如果删除的节点是红黑树节点，则从红黑树中删除
                if (node instanceof HashMap.TreeNode) {
                    ((HashMap.TreeNode)node).removeTreeNode(this, tab, movable);
                }
                //如果是链表节点，且删除的节点为数组下标节点，也就是头节点，直接让下一个作为头。
                else if (node == p) {
                    tab[index] = ((HashMap.Node)node).next;
                }
                //为链表结构，删除的节点在链表中，要把删除的下一个节点设为上一个节点的下一个节点。
                else {
                    p.next = ((HashMap.Node)node).next;
                }

                //修改计数器
                ++this.modCount;
                //长度减1
                --this.size;
                this.afterNodeRemoval((HashMap.Node)node);
                //返回删除的节点
                return (HashMap.Node)node;
            }
        }

        return null;
    }
```
删除还有clear方法，把所有的数组下标元素都置位null。  

### 3.9 get()
下面在看较为简单的获取元素。
```java
    public V get(Object key) {
        HashMap.Node e;
        //也是调用getNode方法来完成
        return (e = this.getNode(hash(key), key)) == null ? null : e.value;
    }

    final HashMap.Node<K, V> getNode(int hash, Object key) {
    	//tab哈希数组，first头节点，n长度，k为key
        HashMap.Node[] tab;
        HashMap.Node first;
        int n;
        //如果哈希数组不为null，且长度大于0，获取key值所在的链表头赋值给first
        if ((tab = this.table) != null && (n = tab.length) > 0 && (first = tab[n - 1 & hash]) != null) {
            Object k;
            //如果是头节点，则直接返回头节点。
            if (first.hash == hash && ((k = first.key) == key || key != null && key.equals(k))) {
                return first;
            }

            HashMap.Node e;
            //结果不是头节点
            if ((e = first.next) != null) {
            	//判断是否是红黑树结构
                if (first instanceof HashMap.TreeNode) {
                	//去红黑树中找，然后返回
                    return ((HashMap.TreeNode)first).getTreeNode(hash, key);
                }

                //是属于链表节点，遍历链表，找到该节点并返回
                do {
                    if (e.hash == hash && ((k = e.key) == key || key != null && key.equals(k))) {
                        return e;
                    }
                } while((e = e.next) != null);
            }
        }

        return null;
    }
```

hashMap源码暂时分析到这里，能力有限，如果内容出现错误，欢迎指出。