---
title: leetcode 104
tags: [leetcode]
date: 2015-12-19 15:06:00
categories: 算法
---

```
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
int maxDepth(struct TreeNode* root) {
    if(root == NULL)
        return 0;
    int x = maxDepth(root->left);
    int y = maxDepth(root->right);
    if(x>y)
        return x+1;
    else return y+1;
}
```