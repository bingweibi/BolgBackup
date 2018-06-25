---
title: Java基础---String相关
date: 2018-04-05 11:16:09
tags: 
	- Java基础
---

# Java中字符串的不变性

一旦一个String对象在内存中被创建出来之后，存储在堆中，这个对象就无法被更改。String类中的所有方法都没有改变这个字符串对象，只是返回了一个新的对象。

```
String s = "abcd";
```
<center>![](http://p2g00vblr.bkt.clouddn.com/String%E4%B8%8D%E5%8F%AF%E5%8F%98%E6%80%A71.jpeg)</center>

```
String s1 = s;
```
<!-- more -->

<center>![](http://p2g00vblr.bkt.clouddn.com/String%E4%B8%8D%E5%8F%AF%E5%8F%98%E6%80%A72.jpeg)</center>

```
s = s.concat("ef");
```

<center>![](http://p2g00vblr.bkt.clouddn.com/String%E4%B8%8D%E5%8F%AF%E5%8F%98%E6%80%A73.jpeg)</center>

如果你需要一个可修改的字符串，应该使用StringBuffer 或者 StringBuilder。否则会有大量时间浪费在垃圾回收上，因为每次试图修改都有新的string对象被创建出来。

# 为什么将String设计成不可变

在Java中`String`是不可变的，不可变指的是这个类的实例一旦被创建变不可以更改。在实例被创建的时候，他的所有信息都会被初始化同时这些信息也不会被改变。下面将从内存、同步性和数据结构的角度来阐述`String`的不变性概念。

## String Pool

在方法区(JVM)中字符串池是一个特殊的区域,当我们在创建一个字符串对像的时候，如果这个字符串已经存在于字符串池中的话，那么将返回现有字符串的引用，而不是去创建新的对象。
例如：
```
String s1 = "abcd";
String s2 = "abcd";
```

<center>![](http://p2g00vblr.bkt.clouddn.com/java-string-pool.jpeg)</center>

## Hashcode

在Java中`String`的`hashcode`的使用频繁.例如在`HashMap、HashSet`。String的不可变性保证了`hashcode`总是一样的，那样的话不用去担心一些不必要的麻烦，同时意味着在每次使用同一个字符串的时候，不用重新计算一次，这样更高效。

## 对于其他类来说使用更加便利

例：

```
HashSet<String> set = new HashSet<String>();
set.add(new String("a"));
set.add(new String("b"));
set.add(new String("c"));

for(String s: set){
    s.value = "a";
}
```

首先String是没有value这个字段的，这里只是为了说明问题。上面如果字符串可以改变，那么就会违背Set的设计原则(Set不允许重复).

## 安全性

String作为参数被广泛的运用于其他类中。例如，网络连接、文件操作等等。如果字符串是可变的，那么网络连接或者文件会被改变，这会造成严重的安全问题。因为当连接的时候，会认为已经连接成功到目标机器，其实并没有(如果可变的话)。同时字符串可变的话也会造成反射的安全问题。

## 不可变对象是线程安全的

因为不可变对象是不可变的，能够在多线程中共享。不需要再进行同步操作。

## 总结

`String`被设计为不可变类的原因，主要是考虑到高效和安全性。这也是大多数情况下首选不可变类的原因。

# 创建String是使用" "还是构造器？

在Java中创建String有两种方式：

```
String s1 = " ";
String s2 = new String("abc");
```

这两种方式有什么不同呢？

1. 利用两个例子解释

例：

```
String a = "abcd";
String b = "abcd";
System.out.println(a == b);  // True
System.out.println(a.equals(b)); // True
```

`a==b` true ：因为两者的引用在方法区中是指向同一个字符串常量的，内存地址也是相同的。

例：

```
String c = new String("abcd");
String d = new String("abcd");
System.out.println(c == d);  // False
System.out.println(c.equals(d)); // True
```

`c==d`false：因为在堆中两者指向不同的对象，不同的对象在内存中的地址是不一样的。

图示：
<center>![](http://p2g00vblr.bkt.clouddn.com/constructor-vs-double-quotes-Java-String.png)</center>

2. intern

```
String c = new String("abcd").intern();
String d = new String("abcd").intern();
System.out.println(c == d);  // Now true
System.out.println(c.equals(d)); // True（1.7之后）
```

## 两者如何使用

如果你只是需要创建一个字符串，可以使用" "方式，但是当你需要在堆中创建一个新的对象时，使用构造函数的方式。

# substring()在1.6和1.7中的不同

## substring(int beginIndex, int endIndex)是干什么的？

```
String x = "abcdef";
x = x.substring(1,3);
System.out.println(x);
```

输出
`bc`

## substring()调用时内存中情况

<cneter>![](http://p2g00vblr.bkt.clouddn.com/substring.jpeg)</center>

## 1.6中的实现情况

<center>![](http://p2g00vblr.bkt.clouddn.com/string-substring-jdk6.jpeg)</center>

当string非常长，而你却需要很短的一段字符串时，这样会造成性能问题。因为你引用了整个字符串（因为这个非常长的字符数组一直在被引用，所以无法被回收，就可能导致内存泄露）。1.6的解决方案是

```
x = x.substring(x, y) + ""
```

## 1.7中实现情况

substring方法会在堆内存中创建一个新的数组

<center>![](http://p2g00vblr.bkt.clouddn.com/string-substring-jdk7.jpeg)</center>

# Java中的Switch对整型、字符型、字符串型的具体实现细节

1.7之后Switch也支持String类型

1. int比较的是整形数值
2. char比较ASCII码
3. string比较`equals()`和`hashcode()`

无论比较什么类型的值，最终都会转化为int。

# 参考链接
https://www.programcreek.com
http://www.hollischuang.com/archives/1330