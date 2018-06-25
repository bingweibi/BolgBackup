---
title: Java集合类---概述
date: 2018-04-21 10:48:05
tags: 
	- Java集合类
---

# Java集合类(UML类图)

先来一张图
<center>![](http://p2g00vblr.bkt.clouddn.com/Java%E9%9B%86%E5%90%88%E7%B1%BB.jfif)</center>

<!--more-->

# 集合类分析

1. 主要分为五大部分
    - List列表
    - Set集合
    - Map映射
    - 迭代器(Iterator,Enumeration)
    - 工具类(Arrays,Collections)
2. Collection接口
    - List列表
        - 元素可以重复
        - 实现类：ArrayList,LinkedList和Vector
        - LinkedList实现Queue接口，可作为队列使用
        - 有序的队列，每个元素都有索引，第一个索引值是0
    - Set集合
        - 元素不可以重复(hashcode和equals保证)
        - 实现类：HashSet和TreeSet
        - HashSet:通过Map中的HashMap实现
        - TreeSet:通过Map中的TreeMap实现
        - TreeSet还实现了SortSet接口(有序)
3. Map接口
    - 实现类(继承AbstractMap)：TreeMap，HashMap和WeakHashMap
    - HahTable:直接实现Map接口
4. Iterator
    - 遍历集合(Collection)的工具
    - ListIterator专门遍历List
5. Enumeration
    - 遍历HashTable，Vector和Stack
6. Arrays和Collections

# 设计模式
Java集合框架中使用的较多的是适配器模式