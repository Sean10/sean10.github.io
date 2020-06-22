---
title: leetcode 258
tags: [leetcode]
date: 2015-12-19 15:02:00
categories: 算法
---

```
int addDigits(int num) {
    return (num != 0 && num%9 == 0)?9:num%9;
}
```