---
title: 剑指offer---03
date: 2018-04-12 17:08:31
tags: 
	- 剑指offer
---

# 题目描述

输入一个链表，从尾到头打印链表每个节点的值。

# 解题思路

1. 遍历给出的链表，直至结尾
2. 将每次遍历的结果放在栈中
3. 便利完成后，将栈中保存的链表节点pop出来
4. 完成输出

# 编码实现

```
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.Stack;
import java.util.ArrayList;

public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        ListNode head = listNode;
        Stack stack = new Stack();
        while (head != null){
            stack.push(head.val);
            head = head.next;
        }
        while (!stack.empty()){
            list.add((Integer) stack.pop());
        }
        return list;
    }
}
```