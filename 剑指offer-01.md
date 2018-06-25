---
title: 剑指offer---01
date: 2018-04-12 16:15:43
tags: 
	- 剑指offer
---

# 题目描述

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

# 解题思路

1. 当然你愿意的话可以暴力去破解，但是一般不建议这样做

2. 
    - 我们可以去开始的点为左下或者右上
    - 这样我们比较的时候可以使用递归的思路
    - 左下(右下)，大于所给的数就往上(左)，小于的话就往右(下)
    - 一直递归，直至找到所给定的值
    - 若没有就返回false
    - 注意边界问题
	
# 代码实现

```
public class Solution {
    public boolean Find(int target, int [][] array) {

        int high = array.length;
        int width = array[0].length;
        int i = high-1,j=0;

        for (;j< width && i>=0;){
            if (array[i][j] > target){
                i--;
            }else if (array[i][j] < target){
                j++;
            }else if (array[i][j] == target){
                return true;
            }
        }
        return false;
    }
}
```