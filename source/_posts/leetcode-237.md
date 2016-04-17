---
title: leetcode 237
tags: [leetcode]
date: 2015-12-19 15:12:00
categories: 算法
---

```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
void deleteNode(struct ListNode* node) {
    struct ListNode* temp = node->next;
    node->val = node->next->val;
    node->next = node->next->next;
    free(temp);
}
```