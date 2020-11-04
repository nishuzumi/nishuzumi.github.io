---
title: "Pow(m,n) O(1) < ⌚️时间复杂度 < O(n)"
date: "2020-11-03 20:29:54"
tags:
categories:
- [算法,中等难度]
---
## 问题1 实现 pow(x, n) ，即计算 x 的 n 次幂函数。
Leetcode链接:[🔗](https://leetcode-cn.com/problems/powx-n/)
示例 1:
>输入: 2.00000, 10
>输出: 1024.00000

示例 2:
>输入: 2.10000, 3
>输出: 9.26100

示例 3:
>输入: 2.00000, -2
>输出: 0.25000
  
解释: $2^{-2}=\frac{1}{2^2}=\frac{1}{4}=0.25$

**说明**

-100.0 < x < 100.0
n 是 32 位有符号整数，其数值范围是 [${−2}^{31}$, $2^{31}−1$] 。

### 思路

假设现在输入x为4，n为5，计算结果应该为`4 * 4 * 4 * 4 * 4 = 1024`。
如果在直接进行`for`的情况下，时间复杂度应该为O(N)

如果将n转化为二进制，结果为`1001`。然后我们写出每个位上的$x^n$的结果。
{% img /images/Pow-m-n-O-1-⌚️时间复杂度-O-n-2020-11-04-16-27-37.png %}

可以看到，当把二进制位上为1的数相乘时，结果为正确答案。也就是`256 * 4 = 1024`

### 代码
  
```javascript
function pow(x,n){
    let mem = x
    let result = 1
    while(n > 0){
        if(n & 2 != 0) result *= mem
        mem *= mem
        n >>= 1
    }
    return result
}
```

## 问题2 实现repeat(str,n)
这个问题是上面问题的变种

示例

>输入：repeat('a',5)
>输出；aaaaa

其实和上面的思路是一样的，都是获得当前的状态值，然后判断是否余数为1，然后把当前状态值加入

### 代码

```javascript
function repeat(str,n){
    let result = ''
    let mem = str
    while(n > 0){
        if (n % 2) result += mem
        mem += mem
        n >>= 1
    }
    return result
}
```

时间复杂度为 O(1) 稍微多一点
空间复杂度为 O(1)