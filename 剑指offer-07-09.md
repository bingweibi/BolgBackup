---
title: 剑指offer---07-09
date: 2018-04-18 15:13:50
tags: 
	- 剑指offer
---

# 题目描述

大家都知道斐波那契数列(0,1,1,2,3,5,8,13....)，现在要求输入一个整数n，请你输出斐波那契数列的第n项。
n<=39

# 解题思路

1. 第一项和第二项没有任何规律，设为初始0,1
2. 第三项 = 第二项 + 第一项
3. 第四项 = 第三项 + 第二项
4. ....
5. f(n) = f(n-1) + f(n-2)

<!--more-->

# 编码实现

```
public class Solution {
    public int Fibonacci(int n) {
        int f1=0,f2=1,result=0;
        if (n == 0){
            return 0;
        }else if (n ==1){
            return f2;
        }
        for (int i = 2; i<=n; i++){
            result = f1+f2;
            f1=f2;
            f2=result;
        }
        return result;
    }
}
```

# 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

# 解题思路

1. 从后往前想，f(n) = 1
2. f(n-1) = 2
3. f(n-2) = f(n) + f(n-1)
4. ...依次递推

# 编码实现

```
public class Solution {
    public int JumpFloor(int target) {
        if (target <= 0) { 
            return -1; 
        } else if (target == 1) { 
            return 1; 
        } else if (target ==2) { 
            return 2; 
        } else { 
            return JumpFloor(target-1)+JumpFloor(target-2); 
        }
    }
}
```

# 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

# 解题思路

1. 从后往前，f(n) = 1
2. f(n-1) = 1+1/2 =2
3. f(n-2) = 1+1+1/1+2/2+1/3 = 4
4. f(n-3) = 1+1+1+1/1+3/3+1/2+2/1+1+2/1+2+1/2+1+1/4 = 8
5. f(n-1) = 2 * f(n)

# 编码实现

```
public class Solution {
    public int JumpFloorII(int target) {
        if (target == 0){
            return -1;
        }else if (target == 1){
            return 1;
        }else {
            return 2*JumpFloorII(target-1);
        }
    }
}
```