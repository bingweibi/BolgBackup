---
title: Java虚拟机---虚拟机字节码执行引擎
date: 2018-04-20 11:33:27
tags: 
	- Java虚拟机
---

# 虚拟机字节码执行引擎

输入的是字节码文件，处理过程是字节码解析的等效过程，输出的是执行结果

## 运行时栈帧结构

1. 栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构(虚拟机栈中)。
2. 栈帧存储着方法的`局部变量表`、`操作数栈`、`动态连接`和`方法返回地址`等信息

<!--more-->

### 局部变量表

1. 一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量
2. 虚拟机通过索引定位的方式使用局部变量表
3. 类变量有两次赋初始值的时候，一次是在准备阶段，赋予系统初始值；另一次是在初始化阶段，赋予程序员定义的初始值。
4. 即使在初始化阶段程序员没有为类变量赋值也没关系，类变量仍然有一个确定的初始值。但是局部变量就不一样，如果一个局部变量定义了但没有赋初始值是不能使用的。
```
public static void main(String[] args){
    int a;
    System.out.println(a)；//编译期间报错，应先进行初始化
}
```

### 操作数栈
### 动态连接
### 方法返回地址

1. 正常完成出口：执行引擎遇到任意一个方法返回的字节码指令，这个时候可能会有返回值传递给上层的方法调用者，是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定
2. 异常完成出口：在方法执行过程中遇到了异常，并且这个异常没有在方法体中得到处理，就会导致方法退出

### 附加信息

## 方法调用
方法调用阶段的任务：确定调用方法的版本(即调用哪一个方法)，暂时还不涉及方法内部的具体运行过程。

### 调用

1. 在所有的方法调用中目标方法在Class文件里面都是一个常量池中的符号引用

2. 在类加载的解析阶段，就会将一部分的**符号引用转化为直接引用**。这种能被解析的前提就是：方法在程序真正运行之前就有一个可确定的调用版本、并且这个方法的调用版本在运行期间不可改变(**编译期可知，运行期不可变**)

3. 接上一条，符合这类要求的方法(非虚方法)：`静态方法 `、`私有方法`、`实例构造器`、`父类方法`和`被final修饰`

### 分派

1. 静态分派
```
public class StaticDispatch{
    static abstract class Human{
    
    }
    static class Man extends Human{
    
    }
    static class Woman extends Human{
    
    }
    public void sayHello(Human guy){
        System.out.println("hello,guy！");
    }
    public void sayHello(Man guy){
        System.out.println("hello,gentleman！");
    }
    public void sayHello(Woman guy){
        System.out.println("hello,lady！");
    }
    public static void main(String[]args){
        Human man=new Man();
        Human woman=new Woman();
        StaticDispatch sr=new StaticDispatch();
        sr.sayHello(man);
        sr.sayHello(woman);
    }
}
```

```
输出：
hello,guy!
hello,guy!
```

解释：
```
Human man = new Man();
```

`Human`称为变量的静态类型，在编译期可知；静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变
`Man`称为变量的实际类型，在运行期才确定
**虚拟机在进行重载的时候是通过参数的静态类型而不是实际类型来判别的**

`静态分派(发生在编译阶段)的典型应用是方法重载`

2. 动态分派(运行期确定，重写)
```
public class DynamicDispatch{
    static abstract class Human{
        protected abstract void sayHello（）；
    }
    static class Man extends Human{
        @Override
        protected void sayHello（）{
        System.out.println（"man say hello"）；
        }
    }
    static class Woman extends Human{
        @Override
        protected void sayHello（）{
        System.out.println（"woman say hello"）；
        }
    }
    public static void main（String[]args）{
        Human man=new Man（）；
        Human woman=new Woman（）；
        man.sayHello（）；
        woman.sayHello（）；
        man=new Woman（）；
        man.sayHello（）；
    }
}
```

```
输出结果：
man say hello
woman say hello
woman say hello
```

3. 单分派与多分派
静态多分派、动态多分派

```
public class Dispatcher {    
    
    static class QQ {    
    }    
    
    static class _360 {    
    }    
    
    public static class Father {    
        public void hardChoice(QQ qq) {    
            System.out.println("father choose qq");    
        }    
    
        public void hardChoice(_360 _360) {    
            System.out.println("father choose 360");    
        }    
    }    
        
    public static class Son extends Father{    
        public void hardChoice(QQ qq) {    
            System.out.println("son choose qq");    
        }    
    
        public void hardChoice(_360 _360) {    
            System.out.println("son choose 360");    
        }    
    }    
    
    public static void main(String[] args) {    
        Father father = new Father();    
        Father son = new Son();    
        father.hardChoice(new _360());    
        son.hardChoice(new QQ());    
            
    }    
    
}  
```
在编译阶段(静态分派)：
1. 一是静态类类型Father还是son，二是方法参数是QQ还是360
2. 产生两条invokevirtual指令，分别指向常量池中Father.hardChoice(360)及Father.hardChoice(QQ)
在运行阶段(动态分派)：
1. 方法的接受者的实际类型是Father还是son

动态语言：它的类型检查主体过程是在运行期而不是编译期