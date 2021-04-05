---
title: leetcode 226
tags: [leetcode]
date: 2015-12-19 19:45:00
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
struct TreeNode* invertTree(struct TreeNode* root) {
    struct TreeNode* temp;
    if(root == NULL)
        return NULL;
    else{
        temp = root->left;
        root->left = invertTree(root->right);
        root->right = invertTree(temp);
        return root;
    }
}
```