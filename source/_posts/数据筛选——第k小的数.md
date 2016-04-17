---
title: 数据筛选——第k小的数
tags: [排序]
date: 2016-01-07 14:52:00
categories: 算法
---

总时间限制: 10000ms 单个测试点时间限制: 5000ms 内存限制: 3000kB
__描述__
小张需要从一批数量庞大的正整数中挑选出第k小的数，因为数据量太庞大，挑选起来很费劲，希望你能编程帮他进行挑选。

__输入__
第一行第一个是数据的个数n(10<=n<=106)，第二个是需要挑选出的数据的序号k(1<=k<=105)，n和k以空格分隔；
第二行以后是n个数据T(1<=T<=109)，数据之间以空格或者换行符分隔。
__输出__
第k小数（如果有相同大小的也进行排序，例如对1,2,3,8,8，第4小的为8，第5小的也为8）。
__样例输入__
```
10 5
1  3  8 20 2 
9 10 12  8 9 
```
__样例输出__
```
8
```

这道题，由于提供的内存限制，1M的记录，都为整型，就达到了4M超出了3M的限制，必须要分组进行排序

```

#include <cstdio>
#include <cstdlib>
#include <cmath>
#define N 200010

int data[N];

int cmp(const void* a,const void* b){
    return *(int *)a-*(int *)b;
}

int main(){
    //freopen("in.txt","r",stdin);
    int n,k;
    scanf("%d%d",&n,&k);
    for(int i = 0;i < k;i++){
        scanf("%d",&data[i]);
    }
    qsort(data,k,sizeof(int),cmp);
    n-=k;
    while(n>0){
        if(n>k){
            for(int i = k;i<2*k;i++){
                scanf("%d",&data[i]);
            }
            qsort(data,2*k,sizeof(int),cmp);
            n-=k;
        }
        else{
            for(int i = k;i < n+k;i++)
                scanf("%d",&data[i]);
            qsort(data,k+n,sizeof(int),cmp);
            n-=n;
        }
    }
    printf("%d\n",data[k-1]);
    return 0;
}

```

参考资料
[1].http://blog.csdn.net/u010663294/article/details/37612219