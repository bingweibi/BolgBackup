---
title: 剑指offer---02
date: 2018-04-12 16:55:49
tags: 
	- 剑指offer
---

# 题目描述
请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

# 解题思路

1.  
    - 首先查找字符串中的空格个数
    - 要替换的字符的长度
    - 扩充原字符串的长度
    - 进行替换
2. 就是采用取巧的replaceAll

# 编程实现

```
(取巧，不要学我)
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	return(str.toString().replaceAll(" ","%20"));
    }
}
```