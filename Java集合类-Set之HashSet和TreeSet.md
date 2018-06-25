---
title: Java集合类---Set之HashSet和TreeSet
date: 2018-04-22 21:08:57
tags: Java集合类
---

# Set HashSet 和 TreeSet简介

1. Set
    - 不包含重复元素的collection
    - 最多只允许一个null 值
2. HashSet 
    - HashSet 实现Set 接口
    - HashSet 不包含重复元素 无序的集合
    - HashSet 最多只允许一个null 值
    - HashSet 非同步 线程不安全
3. TreeSet
    - TreeSet 基于 TreeMap 的 NavigableSet 实现
    - TreeSet 使用元素的自然顺序对元素进行排序，或者根据创建 set 时提供的 Comparator进行排序，具体取决于使用的构造方法
	
<!--more-->
 
# 继承关系

```
public class HashSet<E>
    extends AbstractSet<E>
    
public class TreeSet<E> extends AbstractSet<E>
```

# 实现接口

```
public class HashSet<E> implements Set<E>, Cloneable, java.io.Serializable

public class TreeSet<E> implements NavigableSet<E>, Cloneable, java.io.Serializable
```

# 方法(API)
 和HashMap、HashSet 类似
 
# 源码分析（略）
# 总结

1. HashSet是一个无序的集合，基于HashMap实现；TreeSet是一个有序的集合，基于TreeMap实现
2. HashSet集合中允许有null元素，TreeSet集合中不允许有null元素
3. HashSet和TreeSet都是非同步！在使用Iterator进行迭代的时候要注意fail-fast

# 参考
https://blog.csdn.net/u010648555/article/details/60573808
https://github.com/zxiaofan/JDK/blob/master/JDK1.8/src/java/util/TreeSet.java
https://github.com/zxiaofan/JDK/blob/master/JDK1.8/src/java/util/HashSet.java