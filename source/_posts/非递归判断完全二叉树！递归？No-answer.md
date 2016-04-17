---
title: '非递归判断完全二叉树！递归？No answer!'
tags: [二叉树, 递归]
date: 2015-11-29 18:56:00
categories: 算法
---

判断完全二叉树。

完全二叉树的定义是，只有第n行将所有子节点元素集中在最左以外（可以不满），其余行所有节点均为满。

结构
```
typedef struct tree{
	char data;
	struct tree *lc,*rc;
}BitNode,*BitTree;
```
1.采用层次遍历
通过建立一个队列，将节点an层次入队列。当队列首节点为空时，之后的节点应当均为空，出现非空，则为非完全二叉树
```
bool JudgeComplete(BitTree root)
{
    queue<BitTree> q;
    if(NULL == root)
    	return true;
    q.push(root);
    BitTree cur = NULL;
    bool flag = false;
    while(!q.empty())
    {
    	cur = q.front();
     	q.pop();
             if(cur)
             {
             	if(flag)
             		return false;
                	q.push(cur->lc);
                	q.push(cur->rc);
             }
             else
             	flag = true;
   }
   return true;
}

```

2.递归判断的方法
如果要使用最原始的递归，那么根据分形的原则，完全二叉树并没有处处分形（自相似），所以是肯定不能使用最原始的递归方法，即只传递一个根节点进入的。下面两种传统的递归方法都有漏洞。
那么，可不可以通过追加其他的形参比如*层次*、*深度*、*左右子树标记*来达成目的呢？
假设有一个深度为3的子树，追加这三项是可以判断出的，然而如果深度为100呢？仅一个标记已经无法判断这颗子树的位置，也就无法判断完全二叉树了。
所以，我认为对于完全二叉树的判断，只能采用队列层次遍历的方法
```
bool JudgeComplete(BitTree root)
{
	if(root != NULL){
		if(root->rc != NULL && root->lc == NULL)
			return false;
		JudgeComplete(root->lc);
		JudgeComplete(root->rc);
	} 
	return true;
}
```
存在一个问题，对于同时存在左右子树，不过是在倒数第二行的右子树上同时存在2个子树，而同行的左子树无子树，这种情况也会被判断为完全二叉树

另外还有一个不同的递归算法，同样有比较大的问题
```
bool JudgeComplete(BitTree root)   //采用递归算法来实现判断是否为完全二叉树
{
      if(root == NULL) 
          return   true; 
      if(root->lc == NULL && root->rc == NULL) 
          return   true;  
      if(root->lc  == NULL  && root->rc != NULL || root->lc != NULL && root->rc == NULL) 
          return   false; 
      return   JudgeComplete(root->lc) & JudgeComplete(root->rc); 

}
```
这个算法里就是完全把存在左子树，不存在右子树作为了反例了的，实际上是存在一个这样的二叉树的。