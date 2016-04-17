---
title: 牛的选举——取最大k个数
tags: [链表]
date: 2016-01-08 14:20:00
categories: 算法
---

总时间限制: 1000ms 内存限制: 65536kB
__描述__
现在有N（1<=N<=50000）头牛在选举它们的总统，选举包括两轮：第一轮投票选举出票数最多的K（1<=K<=N）头牛进入第二轮；第二轮对K头牛重新投票，票数最多的牛当选为总统。

现在给出每头牛i在第一轮期望获得的票数Ai（1<=Ai<=1,000,000,000），以及在第二轮中（假设它进入第二轮）期望获得的票数Bi（1<=Bi<=1,000,000,000），请你预测一下哪头牛将当选总统。幸运的是，每轮投票都不会出现票数相同的情况。      

__输入__
第1行：N和K
第2至N+1行：第i+1行包括两个数字：Ai和Bi
__输出__
当选总统的牛的编号（牛的编号从1开始）
__样例输入__
```
5 3
3 10
9 2
5 6
8 4
6 5
```
__样例输出__
```
5
```

```
#include <cstdio>
#include <cstdlib>
#include <cmath>
#include <algorithm>
using namespace std;

#define N 50005

typedef struct node{
    int a,b,num;
}Node;
int ans[N];
int bigger[N];
int smaller[N];
int d[N];

int cmp(const Node &a, const Node &b){
    return  b.a<a.a;
}

int cmp2(const Node &a, const Node &b){
    return b.b<a.b;
}

int main(){
    //freopen("in.txt","r",stdin);
    int n,k;
    scanf("%d%d",&n,&k);
    Node x[N];

    for(int i = 0;i < n;i++){
        scanf("%d%d",&x[i].a,&x[i].b);
        x[i].num = i+1;
    }
    sort(x,x+n,cmp);
    sort(x,x+k,cmp2);

    printf("%d\n",x[0].num);
    return 0;

}
```