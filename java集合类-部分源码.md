---
title: java集合类---部分源码
date: 2018-04-12 22:07:19
tags: 
	- Java集合类
---

# HashMap

## 概述

1. HashMap是基于哈希表的Map接口的非同步实现，Key值和value值都支持使用null值。HashMap不保证映射的顺序，而且不保证映射的顺序一直不变。
2. 通常来说，普通的put和get操作都只需要O(1)的时间复杂度
3. 特别注意的是：HashMap不是同步的，如果有多线程同时去访问一个HashMap,并且其中有至少一个线程会修改map的话，则需要进行同步操作(加锁)。

## 数据结构

1. HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。当发生hash冲突的时候，就是用链表来处理冲突，同一个hash值的链表都存储在一个链表中。
2. 但是当位于一个数组中的元素较多的时候，即hash值相等的元素较多时，通过key值依次查找的效率较低
3. 在JDK1.8中，HashMap中采用数组+链表+红黑树实现，当长度超过阈值8时，将链表转换为红黑树，这样大大较少查找时间。

<!--more-->

## 源码

```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
默认的初始容量是16，且实际的容量必须是2的整数次幂

```
static final int MAXIMUM_CAPACITY = 1 << 30;
```
最大的容量必须是2的幂且小于2的30次方，传入的容量过大的时候将被这个值替换

```
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
默认的装填因子是0.75，如果当前键值对的个数 >= HashMap最大容量*装填因子的时候，将进行rehash操作

```
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
```
JDK1.8中，Entry链表最大长度8，当桶中节点数大于8的时候，将链表转化为红黑树存储
JDK1.8中，当桶中的节点数小于6的时候，将红黑树转为链表存储

在JDK1.6中用Entry描述键值对，在JDK1.8中用Node代替Entry

接下来都是一些方法，没有详细去写。

# HashTable

# 概述

1. HashTable存储的内容是键值对映射的，底层实现是一个Entry数组+链表
2. HashTable和HashMap一样也是散列表，存储的元素也是键值对
3. HashMap允许key和value都为null，但是在HashTable中不可以，会报空指针异常，HashTable的映射不是有序的
4. HashTable的数组默认大小是11，扩容的方式是old*2+1
5. HashMap中数组的大小默认为16，而且一定是2的幂，增加的话为原来的2倍
6. HashTable继承Dictionary类，实现Map接口
7. HashTable大部分的类都用sychronized修饰，所以HashTable是线程安全的
8. fail-fast：快速失败机制就是在并发集合中，其进行迭代操作时，若有其他线程对其进行结构性的修改，这时迭代器会立马感知到，并且会立即抛出ConcurrentModificationException异常，而不是等到迭代完成之后才告诉你出错了。

# ConcurrentHashMap

## 源码

```
private static final int MAXIMUM_CAPACITY = 1 << 30;
```
node数组最大的容量2的30次方

```
private static final int DEFAULT_CAPACITY = 16;
```
默认初始值，必须是2的幂

```
private static final float LOAD_FACTOR = 0.75f;
```
负载因子0.75

```
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
```
链表转树的阈值，如果table[i]下面的链表长度大于8的时候就转化为树
树转链表的阈值，<=6的时候转为链表，仅在扩容tranfer时才可能树转为链表

```
 static class Node<K, V> implements Map.Entry<K, V> {
        final int hash;
        final K key;
        //val和next都会在扩容时发生变化，所以加上volatile来保持可见性和禁止重排序
        volatile V val;
        volatile Node<K, V> next;

        Node(int hash, K key, V val, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return val;
        }
```

# ArrayList

## 概述

1. List接口可以调整大小的数组实现，实现所有可选的List操作，并允许所有元素，包括null，元素可以重复
2. 除了列表接口外，该类提供了一种方法来操作该数组的大小来存储该列表中的数组的大小
3. 时间复杂度
    - size、isEmpty、get、set、iterator和listIterator的调用是常数时间
    - 添加删除的时间复杂度是O(N)，
    - 其他所有操作都是线性时间复杂度
4. 容量：
    - 每个ArrayList都有容量，容量大小至少为List元素的长度，默认为10
    - 容量可以自动增加
    - 如果提前就知道数组的元素较多，可以再添加元素钱通过调用ensureCapacity()方法来提前增加容量以减少后期容量自动增加的开销
    - 当然还可以通过带有初始容量的构造器来初始化这个容量
5. 线程不安全，多线程需同步处理
6. transient:默认的情况下，对象的所有成员变量都将被持久化，在某些情况下，如果你想避免持久化对象的一些成员变量，你可以使用transient关键字来标记它们

## 源码

```
private static final int DEFAULT_CAPACITY = 10;
```
默认容量10

```
/**
     * 扩容，以确保它可以至少持有由参数指定的元素的数目
     *
     * @param minCapacity 所需的最小容量
     */
    private void grow(int minCapacity) {
        // 获取到ArrayList中elementData数组的内存空间长度
        int oldCapacity = elementData.length;
        // 扩容至原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 再判断一下新数组的容量够不够，够了就直接使用这个长度创建新数组，
        // 不够就将数组长度设置为需要的长度
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //若预设值大于默认的最大值检查是否溢出
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 调用Arrays.copyOf方法将elementData数组指向新的内存空间时newCapacity的连续空间
        // 并将elementData的数据复制到新的内存空间
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
1.5倍扩容

```
/**
     * 将ArrayList里面的元素赋值到一个数组中去
     * 如果a的长度小于ArrayList的长度，直接调用Arrays类的copyOf，返回一个比a数组长度要大的新数组，里面元素就是ArrayList里面的元素；
     * 如果a的长度比ArrayList的长度大，那么就调用System.arraycopy，将ArrayList的elementData数组赋值到a数组，然后把a数组的size位置赋值为空。
     *
     * @param a 如果它的长度大的话，列表元素将存储在这个数组中; 否则，将为此分配一个相同运行时类型的新数组。
     * @return 一个包含ArrayList元素的数组
     * @throws ArrayStoreException  将与数组类型不兼容的值赋值给数组元素时抛出的异常
     * @throws NullPointerException 数组为空
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 创建一个新的a的运行时类型数组，内容不变
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

# LinkedList

## 概述

1. LinkedList是List和Deque接口的双向链表的实现。实现了所有可选List操作，并允许包括null值
2. LinkedList既然是通过双向链表来实现的，那么它可以被当做堆栈、队列或者双端队列来进行操作。并且它的顺序访问非常搞笑，而随机访问效率比较低
3. 实现不是同步的，而且也不是线程安全的