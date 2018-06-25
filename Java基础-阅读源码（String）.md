---
title: Java基础---阅读源码（String）
date: 2018-04-04 21:46:43
tags: 	
	- Java基础
---

# 概述

`String Class` 表示的是字符串。在Java中所有的字符串，例如 `“abc”` 都是实现了String这个类的实例。
`String`是常量的，在创建之后它们的值是不可以进行改变的。但是在字符缓冲区是可以进行改变的。因为字符串对象是不可以改变的，所以它们可以被共享。例如：

```
String str = "abc";
```

等同于：

```
char[] data = {'a','b','c'};
String str = new String(data);
```

下面是关于如何使用`String`的例子:

```
System.out.println("abc");
String cde = "cde";
System.out.println("abc" + cde);
String c = "abc".substring(2,3);
String d = cde.substring(1,2);
```

`String`的方法包含了检查序列的每个字符、比较字符串、查找字符串、获得子字符串、复制字符串并进行大小写转化。
Java中提供了对`String`的`+`运算符的支持、将其他对象转化成字符串。`String`的连接是通过`StringBuilder/Stringbuffer`及其append方法实现的。字符串的转化是通过`toString`方法实现的。**String表示一个字符串通过UTF-16(unicode)格式**

<!-- more -->

# 定义

```
public final class More ...String implements java.io.Serializable, Comparable<String>, CharSequence {}
```

从上面可以看出来`String`是被`final`修饰的，所以是不可以更改的，同时还可以看出`String`实现了`Serializable`,`Comparable`,`CharSequence`接口。

# 属性

```
private final char value[];
```

> `final` 修饰,表示不可以更改。例如`String s = "a",s = "a"`,这种情况下s的值并没有修改，只是修改了s的引用、同时可以知道**String的底层是数组实现的。**

```
private int hash;
```
> 缓存string的hash code，默认为0

# 构造方法

1. 使用字符数组、字符串进行构造

```
public More ...String(char value[], int offset, int count) {
         if (offset < 0) {
             throw new StringIndexOutOfBoundsException(offset);
         }
         if (count < 0) {
             throw new StringIndexOutOfBoundsException(count);
         }
         // Note: offset or count might be near -1>>>1.
         if (offset > value.length - count) {
             throw new StringIndexOutOfBoundsException(offset + count);
         }
         this.value = Arrays.copyOfRange(value, offset, offset+count);
     }
```

```
public More ...String(char value[]) {
         this.value = Arrays.copyOf(value, value.length);
     }
```


String的底层是采用字符数组进行实现的，所以我们可以使用一个字符数组来进行创建一个String，当时用字符数组构建Stirng的时候，会使用到`Arrays.copyOf`和`Arrays.copyOfRange`方法，也可以使用字符数组中的某一连续部分进行构建，传入起始和结束即可。这两方法会将原来数组中的内容复制到新创建的`String`中。或者我们使用一个String类型来进行构建String的时候，会将`原String`的`value`和`hash`这两个属性值赋值给`新String`。

2. 使用字节数组构造String

我们知道String的底层是采用字符数组`char[]`实现的,字符数组是使用`unicode`来进行存储的。当采用字节数组来构造String的时候，将字节数组进行解码为字符数组，进而就可以构造新的String。

```
public More ...String(byte bytes[], Charset charset) {
         this(bytes, 0, bytes.length, charset);
         }
```

```
public More ...String(char value[], int offset, int count) {
         if (offset < 0) {
             throw new StringIndexOutOfBoundsException(offset);
         }
         if (count < 0) {
             throw new StringIndexOutOfBoundsException(count);
         }
         // Note: offset or count might be near -1>>>1.
         if (offset > value.length - count) {
             throw new StringIndexOutOfBoundsException(offset + count);
         }
         this.value = Arrays.copyOfRange(value, offset, offset+count);
     }
```

......

3. 使用`StringBuffer` 和 `StringBuider`构造一个`String`

- StringBuffer

```
public More ...String(StringBuffer buffer) {
         synchronized(buffer) {
             this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
         }
     }
```

`StringBuffer`中使用了`Sychronized`,所以效率要低一些。但是是线程安全的。

- StringBuilder

```
public More ...String(StringBuilder builder) {
         this.value = Arrays.copyOf(builder.getValue(), builder.length());
     }
 
     /*
     * Package private constructor which shares value array for speed.
     * this constructor is always expected to be called with share==true.
     * a separate constructor is needed because we already have a public
     * String(char[]) constructor that makes a copy of the given char[].
     */
     More ...String(char[] value, boolean share) {
         // assert share : "unshared not supported";
         this.value = value;
     }
```

# 其他方法

1. charAt(int)
2. compareTo(String)/compareToIgnoreCase(String)
3. concat(String)
4. contains(CharSequence)
5. endsWith(String)
6. equals(Object)
7. indexOf(...)
8. isEmpty()
9. length()
10. spilt()
11. subStirng(int/int,int)
12. toCharArray()
13. trim()

---
charAt(int)
```
public char More ...charAt(int index) {
         if ((index < 0) || (index >= value.length)) {
             throw new StringIndexOutOfBoundsException(index);
         }
         return value[index];
     }
```

compareTo(String)/compareToIgnoreCase(String)
```
public int More ...compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```

concat(String)
```
public String More ...concat(String str) {
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }
```

例如：

```
"cares".concat("s") returns "caress";
 "to".concat("get").concat("her") returns "together";
```

contains(CharSequence)

```
public boolean More ...contains(CharSequence s) {
        return indexOf(s.toString()) > -1;
    }
```

endsWith(String)

```
public boolean More ...endsWith(String suffix) {
        return startsWith(suffix, value.length - suffix.value.length);
    }
```

equals(Object)

```
public boolean More ...equals(Object anObject) {
         if (this == anObject) {
             return true;
         }
         if (anObject instanceof String) {
             String anotherString = (String)anObject;
             int n = value.length;
             if (n == anotherString.value.length) {
                 char v1[] = value;
                 char v2[] = anotherString.value;
                 int i = 0;
                 while (n-- != 0) {
                     if (v1[i] != v2[i])
                         return false;
                     i++;
                 }
                 return true;
             }
         }
         return false;
     }
```

首先`if (this == anObject) `进行判断要比较的对象和当前对象是不是同一个对象，如果是就直接返回true，如果不是在继续进行比较。`anObject instanceof String`如果要比较的对象不是`String`类型的话，直接返回false,如果是的话，再进行比较长度是否相等，相等则一个一个的比较值是否相等，不相等的话直接返回false.

indexOf(...)

```
public int More ...indexOf(int ch, int fromIndex) {
        final int max = value.length;
        if (fromIndex < 0) {
            fromIndex = 0;
        } else if (fromIndex >= max) {
            // Note: fromIndex might be near -1>>>1.
            return -1;
        }

        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
            // handle most cases here (ch is a BMP code point or a
            // negative value (invalid code point))
            final char[] value = this.value;
            for (int i = fromIndex; i < max; i++) {
                if (value[i] == ch) {
                    return i;
                }
            }
            return -1;
        } else {
            return indexOfSupplementary(ch, fromIndex);
        }
    }
```

首先查找的索引和string的长度进行对比，大于则返回-1，否则从头开始进行查找，直到游标等于索引值。索引值<0，设置为==0.

isEmpty()

```
public boolean More ...isEmpty() {
         return value.length == 0;
     }
```

length()

```
public int More ...length() {
         return value.length;
     }
```

spilt()

```
public String[] More ...split(String regex, int limit) {
        /* fastpath if the regex is a
         (1)one-char String and this character is not one of the
            RegEx's meta characters ".$|()[{^?*+\\", or
         (2)two-char String and the first char is the backslash and
            the second is not the ascii digit or ascii letter.
         */
        char ch = 0;
        if (((regex.value.length == 1 &&
             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
             (regex.length() == 2 &&
              regex.charAt(0) == '\\' &&
              (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
              ((ch-'a')|('z'-ch)) < 0 &&
              ((ch-'A')|('Z'-ch)) < 0)) &&
            (ch < Character.MIN_HIGH_SURROGATE ||
             ch > Character.MAX_LOW_SURROGATE))
        {
            int off = 0;
            int next = 0;
            boolean limited = limit > 0;
            ArrayList<String> list = new ArrayList<>();
            while ((next = indexOf(ch, off)) != -1) {
                if (!limited || list.size() < limit - 1) {
                    list.add(substring(off, next));
                    off = next + 1;
                } else {    // last one
                    //assert (list.size() == limit - 1);
                    list.add(substring(off, value.length));
                    off = value.length;
                    break;
                }
            }
            // If no match was found, return this
            if (off == 0)
                return new String[]{this};

            // Add remaining segment
            if (!limited || list.size() < limit)
                list.add(substring(off, value.length));

            // Construct result
            int resultSize = list.size();
            if (limit == 0) {
                while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
                    resultSize--;
                }
            }
            String[] result = new String[resultSize];
            return list.subList(0, resultSize).toArray(result);
        }
        return Pattern.compile(regex).split(this, limit);
    }

```

涉及到正则表达式比较复杂

subStirng(int/int,int)

```
public String More ...substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
```

考虑边界问题，以及起始和终止的大小，之后返回string

toCharArray()

```

    public char[] More ...toCharArray() {
        // Cannot use Arrays.copyOf because of class initialization order issues
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }
```

使用arraycopy进行复制，因为类的初始化问题所以不采用Arrays。copyOf

trim()

```
public String More ...trim() {
        int len = value.length;
        int st = 0;
        char[] val = value;    /* avoid getfield opcode */

        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
```

去除首尾空格

# 参考资料
http://www.hollischuang.com/archives/99
http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/java/lang/String.java#String