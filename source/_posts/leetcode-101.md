---
title: leetcode-101
date: 2017-12-02 17:05:37
tags: [leetcode]
categories: 算法
---

重新复习，这道题想复杂了……

<!--more-->

一开始想到的是层次遍历二叉树，然后用迭代器正反迭代匹配一下，有一个不匹配就返回false。

忘了写中间为空时填充一个nullptr，导致有些案例也返回了true。

又发现如果找不到就填充nullptr，会导致队列永远不会为空，内存溢出MLE的情况，试图在匹配过程中发现空指针，就erase掉，但是迭代过程中erase会导致后续的所有迭代器前移

并且，erase反向迭代器需要进行一下显示类型转换，并在erase时转换成正向迭代器(++it_r).base()。

因为我是new了一个ptr来替代nullptr来实现普遍性，考虑到智能指针可能可以省力，但是实际情况下，似乎容器中不能添加智能指针。虽然我用shared_ptr.get()传递进了容器，但是这样猜测应该是违背容器管理的。

一个容器全部被erase后，迭代器得到的并不是end()，所以需要添加一个empty的判断条件

但是，我又发现，在erase最后一个，之后得到的是地址的顺推，但是end()因为在队列中被释放了2次，所以得到的是原本的倒数第二个，与实际不相符合了。所以需要添加一个计数器来管理结束

看了下参考答案，原来是在遍历的时候就直接正反遍历，最后直接匹配树就好了，我想复杂了。

不过也是成功实现了。

``` c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        deque<TreeNode*> stack_;
        TreeNode* curr = root;

        if(curr)
            stack_.push_back(curr);

        int level = 2;
        int curr_level = 0;

        shared_ptr<TreeNode> ptr_null(new TreeNode(0));
        while(!stack_.empty())
        {
            //cout << curr->val << endl;
            curr = stack_.front();
            stack_.pop_front();

            if(curr->left)
            {
                stack_.push_back(curr->left);
            }else
            {
                stack_.push_back(ptr_null.get());
            }

            if(curr->right)
            {
                stack_.push_back(curr->right);
            }else
            {
                stack_.push_back(ptr_null.get());
            }
            curr_level += 2;

            cout << curr_level << ' '<<level << endl;
            for(deque<TreeNode*>::iterator it_t = stack_.begin(); it_t != stack_.end(); it_t++)
                cout << (*it_t)->val << ' ';
            cout << endl;

            if(level == curr_level)
            {
                curr_level = 0;
                deque<TreeNode*>::iterator it;

                // for(it = stack_.begin(); it != stack_.end(); it++)
                //     cout << (*it)->val << "";
                // cout << endl;

                int cnt = 0;
                deque<TreeNode*>::reverse_iterator it_r;
                int temp_lv = level/2;
                for(it = stack_.begin(), it_r = stack_.rbegin();\
                   cnt < temp_lv; cnt++)
                {
                    // cout << cnt << ' '<< level/4 <<endl;
                    // for(deque<TreeNode*>::iterator it_t = stack_.begin(); it_t != stack_.end(); it_t++)
                    //     cout << (*it_t)->val << "";
                    // cout << endl;

                    if((*it) == ptr_null.get() && ptr_null.get() == (*it_r))
                    {
                        it = stack_.erase(it);
                        it_r = deque<TreeNode*>::reverse_iterator(stack_.erase(next(it_r).base()));
                        level -= 2;
                        continue;
                    }

                    if((*it)->val == (*it_r)->val)
                    {
                        cout << cnt << ' '<<temp_lv << endl;

                        cout << (*it)->val << " " << *it << " "<< (*it_r)->val << " " << *it_r << endl;
                         it++;
                        it_r++;
                        continue;
                    }
                    else
                    {
                        cout << cnt << ' '<< temp_lv << endl;
                        cout << (*it)->val << " " << *it << " "<< (*it_r)->val << " " << *it_r << endl;
                        return false;
                    }
                }
                level *= 2;
            }
        }
        return true;
    }


};
```
