---
title: 二叉树——Huffman树求最短路径和
tags: [二叉树, Huffman]
date: 2015-11-26 14:25:00
categories: 算法
---

[题目链接](http://dsalgo.openjudge.cn/201409week6/1/)

总时间限制: 1000ms 内存限制: 65536kB
####描述
构造一个具有n个外部节点的扩充二叉树，每个外部节点Ki有一个Wi对应，作为该外部节点的权。使得这个扩充二叉树的叶节点带权外部路径长度总和最小：
                                     Min( W1 * L1 + W2 * L2 + W3 * L3 + … + Wn * Ln)
Wi:每个节点的权值。
Li:根节点到第i个外部叶子节点的距离。
编程计算最小外部路径长度总和。
####输入
第一行输入一个整数n，外部节点的个数。第二行输入n个整数，代表各个外部节点的权值。
2<=N<=100
####输出
输出最小外部路径长度总和。
样例输入
```
4
1 1 3 5
```
样例输出
```
17
```

```
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cstring>

typedef struct tree{
    int weight;
    int lc ,rc, parent;
}HufNode,*HufTree;

int GetMin(HufTree H,int k){
    int i = 0,min,minWeight;
    while(H[i].parent != -1){
        i++;
    }
    min = i;
    minWeight = H[i].weight;

    for(;i<k;i++){
        if(minWeight > H[i].weight && H[i].parent == -1){
            minWeight = H[i].weight;
            min = i;
        }
    }

    H[min].parent = 1;
    return min;
}

HufTree Create(HufTree &H,int *s,int n){
    int total = 2*n-1;
    H = (HufTree)malloc(total*sizeof(HufNode));

    for(int i = 0;i < n;i++){
        H[i].weight = *s;
        H[i].lc = -1;
        H[i].rc = -1;
        H[i].parent = -1;
        s++;
    }

    for(int i = n;i < total;i++){
        H[i].weight = 0;
        H[i].lc = -1;
        H[i].rc = -1;
        H[i].parent = -1;
    }

    int min1,min2;
    for(int i = n;i < total;i++){
        min1 = GetMin(H,i);
        min2 = GetMin(H,i);
        H[min1].parent = i;
        H[min2].parent = i;
        H[i].lc = min1;
        H[i].rc = min2;
        H[i].weight = H[min1].weight+H[min2].weight;
    }

    return H;
}

int Trans(HufTree H,int n){
    int ans = 0;

    for(int i = 0;i < n;i++){
        int deepth = 0;
        int temp = i;
        while(H[i].parent != -1){
            deepth++;
            i = H[i].parent;
        }
        i = temp;
        ans += deepth *H[i].weight;
    }
    return ans;
}

int main(){
    //freopen("in.txt","r",stdin);
    int n;
    scanf("%d",&n);
    int s[1000];
    for(int i = 0;i < n;i++){
        scanf("%d",&s[i]);
    }
    HufTree H = Create(H,s,n);
    printf("%d\n",Trans(H,n));
    return 0;
}
```

参考资料：
[1]http://blog.csdn.net/ns_code/article/details/19174553