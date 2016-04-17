---
title: leetcode 260
tags: [leetcode]
date: 2015-12-19 20:27:00
categories: 算法
---

```
/**
 * Return an array of size *returnSize.
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* singleNumber(int* nums, int numsSize, int* returnSize) {
    int xor = 0;
    for(int i = 0;i < numsSize;i++)
        xor ^= nums[i];
    int temp = xor;
    temp &= ~temp+1;
    int ansX = 0;
    for(int i = 0;i < numsSize;i++)
        if(nums[i]&temp)//这一行还有点纳闷，如果换成(nums[i]&temp == temp)就会WA
            ansX ^= nums[i];
    int ansY = ansX^xor;
    *returnSize = 2;
    int *ans;
    ans = malloc(sizeof(int)*2);
    ans[0] = ansX;
    ans[1] = ansY;
    return ans;
}
```