---
title: The Peanuts
tags: [排序]
date: 2016-01-08 17:39:00
categories: 算法
---

总时间限制: 1000ms 内存限制: 65536kB
__描述__
```
Mr. Robinson and his pet monkey Dodo love peanuts very much. One day while they were having a walk on a country road, Dodo found a sign by the road, pasted with a small piece of paper, saying "Free Peanuts Here! " You can imagine how happy Mr. Robinson and Dodo were.

There was a peanut field on one side of the road. The peanuts were planted on the intersecting points of a grid as shown in Figure-1\. At each point, there are either zero or more peanuts. For example, in Figure-2, only four points have more than zero peanuts, and the numbers are 15, 13, 9 and 7 respectively. One could only walk from an intersection point to one of the four adjacent points, taking one unit of time. It also takes one unit of time to do one of the following: to walk from the road to the field, to walk from the field to the road, or pick peanuts on a point.

According to Mr. Robinson's requirement, Dodo should go to the plant with the most peanuts first. After picking them, he should then go to the next plant with the most peanuts, and so on. Mr. Robinson was not so patient as to wait for Dodo to pick all the peanuts and he asked Dodo to return to the road in a certain period of time. For example, Dodo could pick 37 peanuts within 21 units of time in the situation given in Figure-2.

Your task is, given the distribution of the peanuts and a certain period of time, tell how many peanuts Dodo could pick. You can assume that each point contains a different amount of peanuts, except 0, which may appear more than once.
```
__输入__
```
The first line of input contains the test case number T (1 <= T <= 20). For each test case, the first line contains three integers, M, N and K (1 <= M, N <= 50, 0 <= K <= 20000). Each of the following M lines contain N integers. None of the integers will exceed 3000\. (M * N) describes the peanut field. The j-th integer X in the i-th line means there are X peanuts on the point (i, j). K means Dodo must return to the road in K units of time.
```
__输出__
```
For each test case, print one line containing the amount of peanuts Dodo can pick.
```
__样例输入__
```
2
6 7 21
0 0 0 0 0 0 0
0 0 0 0 13 0 0
0 0 0 0 0 0 7
0 15 0 0 0 0 0
0 0 0 9 0 0 0
0 0 0 0 0 0 0
6 7 20
0 0 0 0 0 0 0
0 0 0 0 13 0 0
0 0 0 0 0 0 7
0 15 0 0 0 0 0
0 0 0 9 0 0 0
0 0 0 0 0 0 0
```
__样例输出__
```
37
28
```

其实没用到什么技巧，只是排序。
```
#include <cstdio>
#include <cstdlib>
#include <algorithm>

#define MAX 50

typedef struct node{
    int weight;
    int x,y;
}Node;
int v[MAX][MAX];

int main(){
    //freopen("in.txt","r",stdin);
    int T,M,N,K;
    Node order[2500];
    scanf("%d",&T);
    while(T--){
        int len = 0;
        int currT = 0,ans = 0;;
        scanf("%d%d%d",&M,&N,&K);
        for(int i = 1;i <= M;i++){
            for(int j = 1;j <= N;j++){
                scanf("%d",&v[i][j]);
                if(v[i][j] != 0){
                    int k;
                    for(k = 0;k < len && v[i][j] < order[k].weight;k++);
                    for(int l = len;l > k;l--){
                        order[l] = order[l-1];
                    }
                    order[k].weight = v[i][j];
                    order[k].x = i;
                    order[k].y = j;
                    len++;
                }
            }
        }
        int next_i,next_j,start = 0;
        currT = 0;
        int out_i = 0;
        int curr_i = 0;
        int curr_j = order[start].y;
        next_i = order[start].x;
        next_j = order[start].y;
        int nextT = currT;
        while(start < len){
            out_i = order[start].x;
            nextT += abs(curr_i-next_i)+abs(curr_j-next_j);
            nextT++;
            if(nextT+out_i <= K){
                currT = nextT;
                ans += order[start].weight;
                curr_i = next_i;
                curr_j = next_j;
                if(nextT+out_i == K)
                    break;
            }else{
                nextT = currT;
                break;
            }

            start++;
            next_i = order[start].x;
            next_j = order[start].y;
        }

        printf("%d\n",ans);
    }
    return 0;
}
```