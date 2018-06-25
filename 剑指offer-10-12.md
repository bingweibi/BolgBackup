---
title: 剑指offer---10-12
date: 2018-04-20 19:31:16
tags: 
	- 剑指offer
---

# 题目描述
我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

# 解题思路
斐波拉切数列同样的思路

# 编码实现
```
public class Solution {
    public int RectCover(int target) {
        int f1 = 1,f2 = 2,sum = 0;
        if (target ==0){
            return 0;
        }else if (target == 1){
            return 1;
        }else if (target == 2){
            return 2;
        }
        for(int i=target;i>2;i--){
            sum = f1+f2;
            f1=f2;
            f2=sum;
        }
        return sum;
    }
}
```

<!--more-->

# 题目描述
输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

# 解题思路
参考答案

# 编码实现
```
/**
 * 思路：将n与n-1想与会把n的最右边的1去掉，比如
 * 1100&1011 = 1000
 * 再让++num即可计算出有多少个1
 */
public class Solution {
    public int NumberOf1(int n) {
        int num = 0;
        while (n!=0){
            ++num;
            n=(n-1)&n;
        }
        return num;
    }
}
```

# 题目描述
给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

# 解题思路
1. 指数小于0、指数为0和指数大于0 
2. 指数小于0 返回1.0
3. 指数大于0 循环 1*x（指数递减直至0）
4. 指数小于0，返回1.0/指数大于0的情况

# 编码实现
```
public class Solution {
    public double Power(double base, int exponent) {
        if (Math.abs(exponent) == 0 ){
            return 1.0;
        }else if (exponent < 0){
            return 1.0/(Math.pow(base,Math.abs(exponent))) ;//省略没有用for循环
        }else {
            return Math.pow(base,exponent);
        }
  }
}
```
