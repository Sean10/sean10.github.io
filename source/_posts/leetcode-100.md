---
title: leetcode 100
tags: [leetcode]
date: 2015-12-19 19:24:00
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
bool isSameTree(struct TreeNode* p, struct TreeNode* q) {
    if(p == NULL || q == NULL)
        return p == q;
    else
        return p->val == q->val && isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
}
```