---
title: Java集合类---List之Stack
date: 2018-04-21 20:34:44
tags:
	- Java集合类
---

# Stack简介

```
public
class Stack<E> extends Vector<E>
```

1. Stack 先进后出
2. Stack 数组实现
3. 继承自 Vector ，所以与 Vector 类似

<!--more-->

# 继承关系

`Stack 继承 Vector`

# 实现接口

`Stack 实现 Serializable, Cloneable, Iterable<E>, Collection<E>, List<E>, RandomAccess`

# Stack方法(API)
```
    boolean empty() 
          测试堆栈是否为空。
    E   peek() 
          查看堆栈顶部的对象，但不从堆栈中移除它。
    E   pop() 
          移除堆栈顶部的对象，并作为此函数的值返回该对象。
    E   push(E item) 
          把项压入堆栈顶部。
    int search(Object o) 
          返回对象在堆栈中的位置，以 1 为基数。
          
    其它与Vector类似
```

# Stack源码

```
public
class Stack<E> extends Vector<E> {
    /**
     * Creates an empty Stack.
     */
    public Stack() {
    }

    /**
     * Pushes an item onto the top of this stack. This has exactly
     * the same effect as:
     * <blockquote><pre>
     * addElement(item)</pre></blockquote>
     * stack push操作 
     *
     * @param   item   the item to be pushed onto this stack.
     * @return  the <code>item</code> argument.
     * @see     java.util.Vector#addElement
     */
    public E push(E item) {
        addElement(item);

        return item;
    }

    /**
     * Removes the object at the top of this stack and returns that
     * object as the value of this function.
     * stack pop操作
     *
     * @return  The object at the top of this stack (the last item
     *          of the <tt>Vector</tt> object).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }

    /**
     * Looks at the object at the top of this stack without removing it
     * from the stack.
     * stack peek操作(不移除)
     *
     * @return  the object at the top of this stack (the last item
     *          of the <tt>Vector</tt> object).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }

    /**
     * Tests if this stack is empty.
     * stack 是否非空操作 
     *
     * @return  <code>true</code> if and only if this stack contains
     *          no items; <code>false</code> otherwise.
     */
    public boolean empty() {
        return size() == 0;
    }

    /**
     * Returns the 1-based position where an object is on this stack.
     * If the object <tt>o</tt> occurs as an item in this stack, this
     * method returns the distance from the top of the stack of the
     * occurrence nearest the top of the stack; the topmost item on the
     * stack is considered to be at distance <tt>1</tt>. The <tt>equals</tt>
     * method is used to compare <tt>o</tt> to the
     * items in this stack.
     * stack 查找操作
     *
     * @param   o   the desired object.
     * @return  the 1-based position from the top of the stack where
     *          the object is located; the return value <code>-1</code>
     *          indicates that the object is not on the stack.
     */
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 1224463164541339165L;
}
```
# 总结

1. stack 底层数组实现
2. stack 注意 peek 和 pop
3. stack 继承Vector 继承全部属性和功能

# 参考
https://blog.csdn.net/u010648555/article/details/59705773
https://github.com/zxiaofan/JDK/blob/master/JDK1.8/src/java/util/Stack.java