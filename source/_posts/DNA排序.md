---
title: DNA排序
tags: [排序]
date: 2016-01-08 18:58:00
categories: 算法
---

总时间限制: 1000ms 内存限制: 65536kB
__描述__
```
现在有一些长度相等的DNA串（只由ACGT四个字母组成），请将它们按照逆序对的数量多少排序。
逆序对指的是字符串A中的两个字符A[i]、A[j]，具有i < j 且 A[i] > A[j] 的性质。如字符串”ATCG“中，T和C是一个逆序对，T和G是另一个逆序对，这个字符串的逆序对数为2。
```

__输入__
```
第1行：两个整数n和m，n(0<n<=50)表示字符串长度，m(0<m<=100)表示字符串数量

第2至m+1行：每行是一个长度为n的字符串
```
__输出__
```
按逆序对数从少到多输出字符串，逆序对数一样多的字符串按照输入的顺序输出。
```
__样例输入__
```
10 6
AACATGAAGG
TTTTGGCCAA
TTTGGCCAAA
GATCAGATTT
CCCGGGGGGA
ATCGATGCAT
```
__样例输出__
```
CCCGGGGGGA
AACATGAAGG
GATCAGATTT
ATCGATGCAT
TTTTGGCCAA
TTTGGCCAAA
```

本以为可能在排序之外需要剪枝之类的，事实上用不到，直接用快排就可以过
``` c++
#include <cstdio>
#include <cstdlib>
#include <algorithm>
#include <iostream>
using namespace std;

typedef struct node{
    char str[50];
    int num;
}Node;

int cmp(const Node &a,const Node &b){
    return a.num < b.num;
}

int main(){
    //freopen("in.txt","r",stdin);
    int n,m;
    scanf("%d%d",&n,&m);
    Node s[100];
    for(int i = 0;i < m;i++){
        scanf("%s",s[i].str);
        int num = 0;
        for(int j = 0;j < n;j++){
            if(s[i].str[j] == 'A')
                continue;
            else{
                for(int k = j+1;k < n;k++){
                    if(s[i].str[k] < s[i].str[j])
                        num++;
                }
            }
        }
        s[i].num = num;
    }

    sort(s,s+m,cmp);
    for(int i = 0;i < m;i++)
        printf("%s\n",s[i].str);
    return 0;
}
```