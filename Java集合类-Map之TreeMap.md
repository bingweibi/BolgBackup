---
title: Java集合类---Map之TreeMap
date: 2018-04-22 21:08:23
tags: Java集合类
---

# TreeMap简介

1. TreeMap 一个机遇有序的 k-v 集合，基于红黑树的 NavigableMap 实现
2. TreeMap 根据键自然排序进行排序，也可以根据映射创建是提供的 Comparator 进行排序

# 继承关系

```
class TreeMap<K,V>
    extends AbstractMap<K,V>
```
<!--more-->

# 实现接口

```
class TreeMap<K,V> implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```
# TreeMap方法(API)
```
Map.Entry<K,V> ceilingEntry(K key) 
          返回一个键-值映射关系，它与大于等于给定键的最小键关联；如果不存在这样的键，则返回 null。
 K  ceilingKey(K key) 
          返回大于等于给定键的最小键；如果不存在这样的键，则返回 null。
 void   clear() 
          从此映射中移除所有映射关系。
 Object clone() 
          返回此 TreeMap 实例的浅表副本。
 Comparator<? super K>  comparator() 
          返回对此映射中的键进行排序的比较器；如果此映射使用键的自然顺序，则返回 null。
 boolean    containsKey(Object key) 
          如果此映射包含指定键的映射关系，则返回 true。
 boolean    containsValue(Object value) 
          如果此映射为指定值映射一个或多个键，则返回 true。
 NavigableSet<K>    descendingKeySet() 
          返回此映射中所包含键的逆序 NavigableSet 视图。
 NavigableMap<K,V>  descendingMap() 
          返回此映射中所包含映射关系的逆序视图。
 Set<Map.Entry<K,V>>    entrySet() 
          返回此映射中包含的映射关系的 Set 视图。
 Map.Entry<K,V> firstEntry() 
          返回一个与此映射中的最小键关联的键-值映射关系；如果映射为空，则返回 null。
 K  firstKey() 
          返回此映射中当前第一个（最低）键。
 Map.Entry<K,V> floorEntry(K key) 
          返回一个键-值映射关系，它与小于等于给定键的最大键关联；如果不存在这样的键，则返回 null。
 K  floorKey(K key) 
          返回小于等于给定键的最大键；如果不存在这样的键，则返回 null。
 V  get(Object key) 
          返回指定键所映射的值，如果对于该键而言，此映射不包含任何映射关系，则返回 null。
 SortedMap<K,V> headMap(K toKey) 
          返回此映射的部分视图，其键值严格小于 toKey。
 NavigableMap<K,V>  headMap(K toKey, boolean inclusive) 
          返回此映射的部分视图，其键小于（或等于，如果 inclusive 为 true）toKey。
 Map.Entry<K,V> higherEntry(K key) 
          返回一个键-值映射关系，它与严格大于给定键的最小键关联；如果不存在这样的键，则返回 null。
 K  higherKey(K key) 
          返回严格大于给定键的最小键；如果不存在这样的键，则返回 null。
 Set<K> keySet() 
          返回此映射包含的键的 Set 视图。
 Map.Entry<K,V> lastEntry() 
          返回与此映射中的最大键关联的键-值映射关系；如果映射为空，则返回 null。
 K  lastKey() 
          返回映射中当前最后一个（最高）键。
 Map.Entry<K,V> lowerEntry(K key) 
          返回一个键-值映射关系，它与严格小于给定键的最大键关联；如果不存在这样的键，则返回 null。
 K  lowerKey(K key) 
          返回严格小于给定键的最大键；如果不存在这样的键，则返回 null。
 NavigableSet<K>    navigableKeySet() 
          返回此映射中所包含键的 NavigableSet 视图。
 Map.Entry<K,V> pollFirstEntry() 
          移除并返回与此映射中的最小键关联的键-值映射关系；如果映射为空，则返回 null。
 Map.Entry<K,V> pollLastEntry() 
          移除并返回与此映射中的最大键关联的键-值映射关系；如果映射为空，则返回 null。
 V  put(K key, V value) 
          将指定值与此映射中的指定键进行关联。
 void   putAll(Map<? extends K,? extends V> map) 
          将指定映射中的所有映射关系复制到此映射中。
 V  remove(Object key) 
          如果此 TreeMap 中存在该键的映射关系，则将其删除。
 int    size() 
          返回此映射中的键-值映射关系数。
 NavigableMap<K,V>  subMap(K fromKey, boolean fromInclusive, K toKey, boolean toInclusive) 
          返回此映射的部分视图，其键的范围从 fromKey 到 toKey。
 SortedMap<K,V> subMap(K fromKey, K toKey) 
          返回此映射的部分视图，其键值的范围从 fromKey（包括）到 toKey（不包括）。
 SortedMap<K,V> tailMap(K fromKey) 
          返回此映射的部分视图，其键大于等于 fromKey。
 NavigableMap<K,V>  tailMap(K fromKey, boolean inclusive) 
          返回此映射的部分视图，其键大于（或等于，如果 inclusive 为 true）fromKey。
 Collection<V>  values() 
          返回此映射包含的值的 Collection 视图。
```
# TreeMap源码
```
存储结构
TreeMap的排序是基于key的排序实现的，它的每一个Entry代表红黑树中的一个节点，Entry的数据结构：
static final class Entry<K,V> implements Map.Entry<K,V> {    
     // 键    
     K key;    
     // 值    
     V value;    
     // 左孩子    
     Entry<K,V> left = null;    
     // 右孩子    
     Entry<K,V> right = null;    
     // 父节点    
     Entry<K,V> parent;    
     // 当前节点颜色    
     boolean color = BLACK;    
  
     // 构造函数    
     Entry(K key, V value, Entry<K,V> parent) {    
         this.key = key;    
         this.value = value;    
         this.parent = parent;    
     }    
  
.... 
}  

插入和删除操作，之后都要进行平衡二叉树的操作

public V put(K key, V value) {    
    Entry<K,V> t = root;    
    // 若红黑树为空，则插入根节点    
    if (t == null) {    
    // TBD:    
    // 5045147: (coll) Adding null to an empty TreeSet should    
    // throw NullPointerException    
    //    
    // compare(key, key); // type check    
        root = new Entry<K,V>(key, value, null);    
        size = 1;    
        modCount++;    
        return null;    
    }    
    int cmp;    
    Entry<K,V> parent;    
    // split comparator and comparable paths    
    Comparator<? super K> cpr = comparator;    
    // 找出(key, value)在二叉排序树中的插入位置。    
    // 红黑树是以key来进行排序的，所以这里以key来进行查找。    
    if (cpr != null) {    
        do {    
            parent = t;    
            cmp = cpr.compare(key, t.key);    
            if (cmp < 0)    
                t = t.left;    
            else if (cmp > 0)    
                t = t.right;    
            else   
                return t.setValue(value);    
        } while (t != null);    
    }    
    else {    
        if (key == null)    
            throw new NullPointerException();    
        Comparable<? super K> k = (Comparable<? super K>) key;    
        do {    
            parent = t;    
            cmp = k.compareTo(t.key);    
            if (cmp < 0)    
                t = t.left;    
            else if (cmp > 0)    
                t = t.right;    
            else   
                return t.setValue(value);    
        } while (t != null);    
    }    
    // 为（key-value）新建节点    
    Entry<K,V> e = new Entry<K,V>(key, value, parent);    
    if (cmp < 0)    
        parent.left = e;    
    else   
        parent.right = e;    
    // 插入新的节点后，调用fixAfterInsertion调整红黑树。    
    fixAfterInsertion(e);    
    size++;    
    modCount++;    
    return null;    
}    
....
```
# 总结

1. TreeMap 根据key进行排序，排序和定位需要依赖比较器或者覆写Compareable接口，不需要重写equals 和 hashCode方法，就可以去重
2. TreeMap 的查询、插入和删除操作的效率都没有HashMap 高
3. TreeMap 中的key 不能为null

# 参考
https://blog.csdn.net/ns_code/article/details/36421085
https://github.com/zxiaofan/JDK/blob/master/JDK1.8/src/java/util/TreeMap.java