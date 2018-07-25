---
title: 几个简单而又有趣的Python算法题
date: 2018-07-24 13:47:08
tags: python 算法 leetcode
---
## 1. 前言

作为一个合格的程序员，如果仅仅只是对工具或者框架熟悉，而不清楚算法，这肯定不是一个合格的，能够经得起时间考验的程序员。如果把程序员比作是一个武林高手，那么他的算法能力就是他的内功，只有内功修炼深厚了，学习框架，语言或者工具等才会快，并且能够真正的理解他们的用法。作为一个半路出家的低端程序员，为了锻炼自己的算法能力，于是决定在leetcode上刷题来提高自己。就这段时间以来我所遇到的一些有趣而又典型的算法题来跟大家做一个分享。

## 2.推理题

抓了a,b,c,d四名犯罪嫌疑人，其中有一人是小偷，审讯中：

* a说我不是小偷；
* b说c是小偷；
* c说小偷肯定是d；
* d说c胡说！

其中有三个人说的是实话，一个人说的是假话，请编程推断谁是小偷（用穷举法和逻辑表达式）。
这个题其实不难，很适合用来锻炼自己编码解决实际问题的能力。
```
def thief_is():
    for thief in ('a', 'b', 'c', 'd'):
        sum = ('a' != thief) + (thief == 'c') + \
            (thief == 'd') + (thief != 'd')
        if sum == 3
            print("thief is %s"%thief)
```
最后输出的结果为：thief is c
个人觉得这种解法真是非常的巧妙，仿佛就是天然为这道题目而生，让人感觉非常舒服。
<!--more-->

## 3. 数组算法题

### a.两数之和

给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。
你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。示例:
```
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```
本题有两个思路：
* 使用双重循环，遍历每两个元素两两相加的结果，通过判断其和是否满足等于指定数字从而找出这两个元素。思路简单易懂，但是双重循环的时间复杂度为f(n<sup>2</sup>),效率较慢；
* 使用减法，将指定数字减去数组中的任一元素，判断所得的差是否在数组中，通过该元素的值来得到该元素的索引。只有一重循环，效率有所提高，但难点在于通过元素值来得到元素索引。

根据第一种思路：
```
def twosum(nums, target):
    l = len(nums)
    for i in range(l-1):
        for j in range(i+1, l):
            if nums[i]+nums[j] == target:
                return [i, j]
```
(用时7312ms)
根据第二种思路：
```
def twosum(nums, target):
    l = len(nums)
    for i in range(l):
        another = target - nums[i]
        if another in nums:
            j = nums.index(another)
            if i==j:
                continue
            else:
                return [i,j]]
```
(用时1412ms)
当然还有大牛用的更优的解法，也是第二种思路：
```
def twosum(nums, target):
    d={}
    n=len(nums)
    for x in range(n):
        a=target-nums[x]
        if nums[x] in d:
            return d[nums[x]],x
        else:
            d[a]=x
```

### b.有效的数独

判断一个 9x9 的数独是否有效。只需要根据以下规则，验证已经填入的数字是否有效即可。

    数字 1-9 在每一行只能出现一次。
    数字 1-9 在每一列只能出现一次。
    数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。

![PtfOsA.png](https://s1.ax1x.com/2018/07/26/PtfOsA.png)

上图是一个部分填充的有效的数独。

数独部分空格内已填入了数字，空白格用 '.' 表示。

示例 1:
```
输入:
[
  ["5","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
]
输出: true
```
此题比较复杂，判断的条件多，而且是对二元素组进行操作，要求对数组的操作非常熟悉，当然，用python会相对而言更加简单一点。
解法如下：
```
def isValidSudoku(board):
    # 判断一行是否有效
    for i in range(9):
        for j in board[i]:
            if j != '.' and board[i].count(j) > 1:
                return False
        # 判断一列是否有效
        column = [k[i] for k in board]
        for n in column:
            if n != '.' and board[i].count[n] > 1:
                return False
    # 判断九宫格是否有效
    for i in range(3):
        for j in range(3):
            grid = [tem[j*3:(j+1)*3] for tem in board[i*3:(i+1)*3]]
            merge_str = grid[0] + grid[1] + grid[2]
            for m in merge_str:
                if m != '.' and merge_str.count(m) > 1:
                    return False
    return True
```
也有大牛比较pythonic的解法：
```
def isValidSudoku(board):
    seen = sum(([(c, 1), (j, c), (i//3, j//3, c)] \
    for i, row in enumerate(board) for j,c in \
    enumerate(row) if c != '.'), [])
    return len(seen) == len(set(seen))
```
此解法虽然代码量少，但效率却并不高。

### c.旋转图像

给定一个 n × n 的二维矩阵表示一个图像。将图像顺时针旋转 90 度。说明：你必须在原地旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要使用另一个矩阵来旋转图像。

示例 :
```
给定 matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

原地旋转输入矩阵，使其变为:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

