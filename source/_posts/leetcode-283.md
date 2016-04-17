---
title: leetcode 283
tags: [leetcode]
date: 2015-12-19 19:17:00
categories: 算法
---

```
void moveZeroes(int* nums, int numsSize) {
    //int zeros = 0;
    for(int i = numsSize-1;i >= 0;i--){
        if(nums[i] == 0){
            for(int j = i;j < numsSize-1;j++){
                nums[j] = nums[j+1];
            }
            nums[numsSize-1] = 0;
            //zeros++;
        }
    }
}
```