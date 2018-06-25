---
title: Java基础---泛型
date: 2018-04-25 18:38:42
tags: Java基础
---

# 一个问题

```
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();

System.out.println(l1.getClass() == l2.getClass());
```

```
输出：true
```
ok ,反正我在写这个文章之前是不懂的，可能半桶水晃得太厉害了吧！

<!--more-->

# 泛型介绍

泛型：参数化类型，通俗的讲就是就是将类型作为参数传递给一个类或者方法

## 类型参数化

### 没有泛型之前

```
private static class Test {
        Object value;

        public Object getValue() {
            return value;
        }

        public void setValue(Object value) {
            this.value = value;
        }
    }
```

```
Test test = new Test();
test.setValue("123456");
String value = (String) test.getValue();
```

在这里我们使用`Object`来修饰 `value`，这样的话，value就可以是任何类型，但是当我们`get`的时候需要进行类型强转

### 有了泛型之后

```
private static class Test<T> {
        T value;

        public T getValue() {
            return value;
        }

        public void setValue(T value) {
            this.value = value;
        }
    }
```
```
Test<String> test = new Test<>();
test.setValue("123456");
String value = test.getValue();
```
这里我们并没有指定`Object`来修饰value,而是使用`T`，当我们在初始化的时候，指定了`Test<String> test = new Test<>();`.之后`get`的时候就不需要进行强转操作。

但是需要注意
```
Test<String> test = new Test<>();
test.setValue("123456");
String value = test.getValue();
test.setValue(123456);//异常
```
最后一行语句是会报异常的，因为你在前面已经指定了`value`的类型为`String`,后面你设置`test.setValue(123456);`就会报异常

结论：

1. 当具体的类型确定后，泛型又提供了一种类型检测的机制，只有相匹配的数据才能正常的赋值，否则编译器就不通过。
2. 泛型提高了程序代码的可读性，不必要等到运行的时候才去强制转换，在定义或者实例化阶段，因为 Cache<String> 这个类型显化的效果，程序员能够一目了然猜测出代码要操作的数据类型

# 泛型的定义和使用

泛型的3种使用情况
1. 泛型类
2. 泛型方法
3. 泛型接口

## 泛型类

### 泛型类定义

```
private static class Test<T> {
        T value;
}
```
如果一个类被`<T>`这种形式定义的话，那么就成这个类为泛型类

### 泛型类使用

`Test<String> test1 = new Test<>();`

泛型还可以接受多个类型参数

```
public class Test <E,T>{
    E value1;
    T value2;

    public E getValue1(){
        return value1;
    }

    public T getValue2(){
        return value2;
    }
}
```

## 泛型方法

### 泛型方法的定义

```
public <T> setValue(T value) {
            this.value = value;
}
```

`<T>` 代表的是返回类型，T称为类型参数
`T value` T 称为参数化类型，不是运行时真正的参数

### 泛型类与泛型参数同时存在的情况

```
public class Test1<T>{

    public  void testMethod(T t){
        System.out.println(t.getClass().getName());
    }
    public  <T> T testMethod1(T t){
        return t;
    }
}
```
`Test1<T>`泛型类
`testMethod(T t)`普通方法
`<T> T testMethod1(T t)`泛型方法，以自己定义的类型参数为准

```
Test1<String> t = new Test1();
t.testMethod("generic");
Integer i = t.testMethod1(new Integer(1));
```
泛型类的实际类型参数是`String`
泛型方法的实际类型参数是`Integer`
但是，不建议这样写

## 泛型接口

与泛型方法类似

# 通配符 ？

## 为什么要有通配符 ?

```
class Father{}
class Son extends Father{}
Son son = new Son();
Father father = son;  
```

```
List<Son> sonList = new ArrayList<>();
List<Father> fatherList = sonList;
```
上面的代码编译器不通过

**通配符测出现就是为了制定泛型中的类型范围**

通配符的三种形式：
1. `<?>` 被称作无限定的通配符
2. `<? extends T>` 被称作有上限的通配符
3. `<? super T>` 被称作有下限的通配符

### 无限定通配符
```
 public void setValue(Collection<?> collection) {
//            this.value = value;
            collection.add(123);//报异常
            collection.size();
        }
```

`<?>`只提供了只读功能，只具备与具体类型无关的能力

### 有上限的通配符


```
 public void setValue(Collection<? extends Base> collection) {
//            this.value = value;
            collection.add(123);//报异常
            collection.size();
        }
```
这里我们知道`collection` 的类型`Collection`接受`Base`以及`Base`的子类

### 有下限的通配符

```
 public void setValue(Collection<? super Son> collection) {
//            this.value = value;
            collection.add(new Son());
            collection.size(new Father());//不通过
        }
```

**用通配符做的事情，都可以采用类型参数代替**

# 类型擦除

**泛型信息只存在于代码编译阶段，在进入 `JVM` 之前，与泛型相关的信息会被擦除掉，专业术语叫做类型擦除**

```
public static void main(String[] args) {
        List<String> l1 = new ArrayList<>();
        List<Integer> l2 = new ArrayList<>();

        System.out.println(l1.getClass().getName());
        System.out.println(l2.getClass().getName());
        System.out.println(l1.getClass() == l2.getClass());
    }
```
输出
```
java.util.ArrayList
java.util.ArrayList
类型擦除之后都是 java.util.ArrayList
true
```

**在泛型类被类型擦除的时候，之前泛型类中的类型参数部分如果没有指定上限，如 <T> 则会被转译成普通的 Object 类型，如果指定了上限如 <T extends String> 则类型参数就被替换成类型上限**

## 注意
```
List<Integer>[] li2 = new ArrayList<Integer>[10];//报错
List<Boolean> li3 = new ArrayList<Boolean>[2];//报错
```
**Java 不能创建具体类型的泛型数组**

```
List<?>[] li2 = new ArrayList<?>[10];
List<?>[] li3 = new ArrayList<?>[2];
```