---
title: Java虚拟机---虚拟机类加载机制
date: 2018-04-19 22:01:04
tags: 
	- Java虚拟机
---

# 虚拟机类加载机制

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型。这，就是虚拟机的类加载机制。

运行期完成：类型的加载、连接和初始化过程

## 类加载的时机

1. 类从加载内存到卸载内存一共会经历7个阶段:加载-->验证-->准备-->解析-->初始化-->使用-->卸载
2. 类加载经历的阶段：加载-->验证-->准备-->解析-->初始化
3. 类加载：加载-->连接(验证-->准备-->解析)-->初始化-->使用-->卸载
4. 解析在某种情况下可以再初始化之后再进行(为了支持Java语言的运行时绑定)

<!--more-->

## 类加载过程中的初始化时机

 1. 在运行过程中遇到以下字节码指令时，如果类没有初始化，那就要进行初始化：new getstatic putstatic invokestatic
    - 通过new创建对象
    - 读取一个类的静态成员变量（不包含final修饰）
    - 设置一个类的静态成员变量（不包含final修饰）
    - 调用一个类的静态成员函数
2. 进行反射调用的时候，没有类初始化就，那就需要初始化
3. 初始化一个类的时候，如果父类没有初始化就先初始化父类，再初始化本类
4. 虚拟机启动时，虚拟机先初始化主类
5. 使用JDK1.7动态语言时，如果一个java.lang.invoke.MethodHandle实例最后解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化

**类与接口的真正区别在于：当一个类在初始化时，要求其类全部都已经初始化过了，但是一个接口在初始化时，并不要求其父接口全部都完成了初始化，只有在真正使用到父接口的时候(如接口中定义的常量)才会初始化**

## 类加载的过程

**加载-->验证-->准备-->解析-->初始化**

### 加载
 
加载的阶段，虚拟机需要完成3件事情
1. 通过一个类的全限定名来获取定义此类的二进制字节流
2. 将这个字节流所代表的的静态储存结构转化为方法区的运行时的数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

### 验证

目的：确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全

四个阶段的检验动作
1. 文件格式的验证(基于二进制字节流)
2. 元数据的验证(基于方法区的存储结构)
3. 字节码验证(基于方法区的存储结构)
4. 符号引用验证(基于方法区的存储结构)

### 准备

准备阶段是正式为`类变量分配内存`并`设置类变量初值`的阶段，这些变量所使用的的内存将在**`方法区`**中进行分配。

**注意**
1. 这个时期进行内存分配的仅仅只包括类变量(被static修饰的变量)，而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在Java堆中
2. 这个地方说的设置类变量的初始值是指数据类型的零值(0，null，false......)

例如：`public static int value = 123`，这个阶段过后`value`的值是0，而不是123（编译之后才是）
3. 如果类字段的字段属性表中存在`ConstantValue`属性，那么被修饰的值就是指定的值

例如：`public static final int value = 123`,准备阶段之后`value`的值就是123

### 解析

解析阶段就是虚拟机将常量池内的符号引用替换成直接引用的过程

1. 除invokedynamic指令外，虚拟机可以对第一次解析的结构进行缓存，从而避免解析动作重复进行
2. 虚拟机需要保证对同一个实体来说，如果第一次解析成功，以后会一直成功，如果第一次失败的话，那么之后会一直失败

### 初始化

在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划去初始化类变量和其他资源。

静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态域局块可以赋值，但是不能访问
```
public class Test { 
    static{
        i = 0;                   //给变量赋值可以正常编译通过
        System.out.print(i);     //编译器提示"非法向前引用"
    }
    static int i =1;
}
```
```
static class Parent{
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent{
    public static int B = A;
}

public static void main(String[] args){
    System.out.println(Sub.B);
}

输出 2
```

## 类加载器

在类加载阶段中的"通过一个类的全限定名来获取描述此类的二进制字节流"这个动作放在Java虚拟机外部去实现，实现这个动作的代码模块称为"类加载器"。

### 类与加载器

对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立骑在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名字空间.简单来说再进行`equals`的时候，如果这两个类不是由同一个类加载器加载的话，那样就没有比较的意义。

### 双亲委派类型

1. 启动类加载器
2. 其他的类加载器
    - 扩展类加载器
    - 应用程序类加载器

自定义类加载器-父->应用程序类加载器-父->扩展类加载器-父->启动类加载器

双亲委派模型的工作流程:
1. 如果一个类加载器收到了类加载器的请求
2. 它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是这样
3. 因此所有的加载请求最终都应该传送到顶层的启动类加载器中，
4. 只有当父加载器反馈自己无法完成这个加载请求时
5. 子加载器才会尝试自己去加载