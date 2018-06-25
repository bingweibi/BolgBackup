---
title: 剑指offer---05
date: 2018-04-16 19:55:30
tags: 
	- 剑指offer
---

# 题目描述

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

# 解题思路

1. push操作：栈1用来push
2. pop操作：栈1 pop-->栈2push-->栈2pop

# 编码实现
```
import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push(node);
    }
    
    public int pop() {
        if (stack2.empty()){
            if (stack1.empty()){
                System.out.println("空队列");
                return -1;
            }else {
                while (!stack1.empty()){
                    stack2.push(stack1.pop());
                }
            }
        }
        return stack2.pop();
    }
}
```