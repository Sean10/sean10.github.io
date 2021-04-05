---
title: leetcode292
tags: [leetcode]
date: 2015-12-19 14:50:00
categories: 算法
---

这道题传统递归在leetcode上会超时，不过我觉得传统思路还是可以保留的。

数学方法
```
bool canWinNim(int n) {
    return n%4;
}
```

传统递归
```
bool canWinNim(int n) {
    if(n > 0 && n <= 3)
        return true;
    if(canWinNim(n-1)&&canWinNim(n-2)&&canWinNim(n-3))
        return false;
    else return true;
}
```