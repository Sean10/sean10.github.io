---
title: 离散数学——hamming码最小距离
tags: [离散数学, 校验码]
date: 2015-11-15 00:12:00
categories: 专业
---

1：给定H(读取文件方式，第一行两个整数m,n，第二行 m\times (n-m)个0或1，也就是矩阵H的上半部分，下半部单位矩阵自行生成)，计算群码编码函数e_H。计算该编码函数能检测到多少位错误，交互输出字的码字。

输入文件：in.txt，示例：第一行两个整数，第二行累计mxr个整数。所有整数都用一个空格分隔。

3 5

1 0 1 0 0 1

无输出文件。

-----------------------------

2：针对(8,12)编码e，找出最小距离最大的群码编码函数，输出H及最小距离。

无输入文件

输出文件：out.txt，示例，矩阵按行输出

1 1 0 0

0 0 1 0 

....

0 1 0 1

(以上总共256行，此行为说明，程序不输出)

H的最小距离是：3

--------------------------------------------------

以下是代码
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int int_pow(int x,int e){
    int ans = 1;
    while(e--)
        ans*=x;
    return ans;
}

void GetBinary(int* mb,int m, int i){
    for(int j = m;j >= 1;j--){
        mb[j] = i%2;
        i/=2;
    }
}

void CntBinary(int *mb, int n ,int &min){
    int cnt = 0;
    for(int i = 1;i <= n;i++){
        if(mb[i] == 1)
            cnt++;
    }
    if(cnt < min)
        min = cnt;
}

int main(){
    //freopen("in.txt","r",stdin);
    FILE *fp = fopen("in.txt","r");
    if(fp == NULL) printf("ERROR OPEN.\n");
    int m,n;
    int ma[100][100];
    fscanf(fp," %d %d",&m,&n);
    for(int i = 1;i <= m;i++){//行列互换
        for(int j = 1;j <= n-m;j++){
            fscanf(fp," %d",&ma[i][j]);
        }
    }
    /*
    for(int i = 1;i <= m;i++){
        for(int j = 1;j <= n-m;j++)
            printf("%d",ma[i][j]);
        printf("\n");
    }
    */
    int mb[100];
    int mc[100];
    int min = m;
    for(int i = 1;i < int_pow(2,m);i++){
        GetBinary(mb,m,i);
        for(int j = m+1;j <= n;j++)
            mc[j] = 0;
        for(int j = 1;j <= m;j++)
            mc[j]=mb[j];
        for(int j = m+1;j <= n;j++){
            for(int k = 1;k <= m;k++){
                mc[j] += mb[k]*ma[k][j-m];
            }
            mc[j] %= 2;
        }
        CntBinary(mc,n,min);
    }
    printf("The minimum hamming distance is:%d\n",min);

    for(int i = 1;i <= m;i++)
        scanf(" %d",&mb[i]);
    for(int i = 0;i <= n;i++)
        mc[i] = 0;
    for(int i = 1;i <= m;i++)
        mc[i]=mb[i];

    for(int i = m+1;i <= n;i++){
        for(int j = 1;j <= m;j++){
            mc[i] += mb[j]*ma[j][i-m];
        }
        mc[i] %= 2;
    }

    printf("The code word is:");
    for(int i = 1;i <= n;i++)
        printf("%d",mc[i]);
    printf("\n");

    return 0;
}
```

第二题这里，依照以下定理自己手算奇偶校验矩阵就比较简单了，最小距离最大的充要条件是奇偶校验矩阵任意两行线性无关。

定理是(n,k)线性分组码的最小Hamming距离为d的充要条件是，H矩阵中任意d-1列线性无关。
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define N 256

int int_pow(int x,int e){
    int ans = 1;
    while(e--)
        ans*=x;
    return ans;
}

void GetBinary(int* mb,int m, int i){
    for(int j = m;j >= 1;j--){
        mb[j] = i%2;
        i/=2;
    }
}

void CntBinary(int *mb, int m ,int &min){
    int cnt = 0;
    for(int i = 1;i <= m;i++){
        if(mb[i] == 1)
            cnt++;
    }
    if(cnt < min)
        min = cnt;
}

int main(){
    FILE *fp=fopen("out.txt","w");
    int m = 8,n = 12;
    int ma[N][5]={{0,0,0,0,0},
              {0,1,1,0,0},
              {0,0,1,1,0},
              {0,1,1,0,1},
              {0,1,1,1,0},
              {0,1,0,0,1},
              {0,0,1,0,1},
              {0,1,0,1,0},
              {0,0,0,1,1}};
    /*
    for(int i = 0;i < 8;i++){
        for(int j = 0;j < 4;j++){
            printf("%d",H[i][j]);
        }
        printf("\n");
    }
    */
    int mb[N];
    int mc[N];
    int min = m;
    for(int i = 1;i < int_pow(2,m);i++){
        GetBinary(mb,m,i);
        for(int j = m+1;j <= n;j++)
            mc[j] = 0;
        for(int j = 1;j <= m;j++)
            mc[j]=mb[j];

        for(int j = m+1;j <= n;j++){
            for(int k = 1;k <= m;k++){
                mc[j] += mb[k]*ma[k][j-m];
            }
            mc[j] %= 2;
        }
        for(int j = m+1;j <= n;j++)
            fprintf(fp,"%d\t",mc[j]);
        fprintf(fp,"\n");

        CntBinary(mc,n,min);
    }
    fprintf(fp,"The maximum of all the minimum distance is:%d\n",min);
    return 0;
}

```