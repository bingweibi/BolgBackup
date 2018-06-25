---
title: Java基础---Integer
date: 2018-04-05 22:28:22
tags: 
	- Java基础
---

# 定义

`Integer`类将Java中的原始类型`int`的值包装在对象中。Integer类型的对象包含一个int类型的字段。

`Integer`类提供将`int`与`string`互相转换的方法，另外还提供有用在的处理int型时的常量和方法。

```
public final class More ...Integer extends Number implements Comparable<Integer> {}
```

1. final修饰，不能被继承
2. 继承Number
3. 实现Comparable，可以利用comparable to进行比较(同类型之间)

# 属性

## 私有属性

```
private final int value;
private static final long serialVersionUID = 1360826667806852920L;
```

## 公有属性

```
public static final int   MAX_VALUE = 0x7fffffff;
public static final int   MIN_VALUE = 0x80000000;
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
public static final int SIZE = 32;
public static final int BYTES = SIZE / Byte.SIZE;
```
# 方法

## 构造方法

```
//表示指定的 int 值
public More ...Integer(int value) {
         this.value = value;
     }
```

```
//表示指定的 string 值,利用parseInt
public More ...Integer(String s) throws NumberFormatException {
         this.value = parseInt(s, 10);
     }
```

## byteValue()...

```
//Integer转化为byte返回
public byte More ...byteValue() {
         return (byte)value;
     }
```

## toString()

```
//返带符号回十进制的数值并作为字符串返回
public String More ...toString() {
         return toString(value);
     }
```

## hashcode()

```
public int More ...hashCode() {
         return Integer.hashCode(value);
     }
```

## equals()

```
//与指定类型进行比较
public boolean More ...equals(Object obj) {
         if (obj instanceof Integer) {
             return value == ((Integer)obj).intValue();
         }
         return false;
     }
```

## ...

# Integer 和 int 的关系

1. int是java提供的8种基本类型之一。Java为每个原始类型提供了封装类，Integer是java为int提供的封装类。
2. int的默认值是0，Integer的默认值是null
3. 两者都可以表示一个数值
4. 两者不可以互用，因为它们是两种不同的数据类型。

