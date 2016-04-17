---
title: huffman编码实现压缩与解压缩
tags: [Huffman, 压缩]
date: 2015-12-13 21:23:00
categories: 专业
---

题目：将任意一个指定的文件进行哈夫曼编码，并以真正的二进制位生成一个二进制文件（压缩文件）；反过来，可将一个压缩文件解码还原为原来的文件。

以下是编码过程中需要注意的地方
1.读入字符
这里需要明白fread的运用。这段代码要实现的功能是对各类型文件进行转码，所以文本输入的方式fscanf不能在这里使用，只能用fread.
读入过程中需要记录文件中总计的单字节字符数量n，后面需要写入编码的文件中用于后续解码时判断是否已经到了最后一个字符的编码处，跳过补充的0代码段的解码。

2.统计字符出现频次
这里需要统计实际的哈弗曼树的叶子节点的数量，即不同的字符数量。

3.通过2的过程已经得到了每个字符的权值，即前nReal项已经输入完毕，可以开始建树
首先通过Select函数从已经得到的字符中提取最小的2个字符的权值，然后进行循环建树。这里的过程就是常规的建立静态分配空间以后的哈夫曼树的过程。

4.建树完毕，接下来开始进行编码操作
主体编码的流程是左子树为0，右子树为1，如果该节点两个孩子节点均指向0,,则该节点编码完毕，将临时分配的cd空间内的字符存入哈夫曼树中。

5.编码完毕，接下来直接输出即可
重新按照文件的字符出现的顺序，找到该字符对应的编码，按每8位完成一个编码写入文件，当抵达最后一个字符的时候，如果不足8位，补充编码，后面解码的时候的n就是用在这里跳过补充的编码的。

注意：这里并不能直接输出
如果直接以char型写入，输出的并没有起到压缩的作用，最多只能算是转码，所以这里需要追加一个过程，使得输出的0、1为确实的单比特输出，而不是1个字节的字符型输出。
并且这里char型与unsigned char型在首位也是存在差别的，所以建议转换类型为unsigned char。

下面解码
虽然说哈弗曼编码运用了前缀编码的原理，编码不会发生无法解码的过程，但是其终究是建立在具有了哈弗曼树的前提下才可进行解码。假如没有哈夫曼树，因为文件内部的二进制码是没有间断点的，如果一个一颗成形的哈夫曼树，我们并不能判断是否已经到了这个字符的编码结束的位置。假如迭代暴力解码，间断点的可能性有(n-1)!种，相当于o(n^n)级的复杂度，不可行。

所以我们在这里在编码时就将整棵树输出到解码文件的首部，解码时通过读取此树，再根据这棵树进行解码。

1.首先，读取nReal，动态分配足够的空间给接下来建立的树，读取文件中关于树的data和权值。

2.建树完毕，读取二进制编码，根据01决定左、右子树，如果抵达叶子节点，则其左右子树均为0，存入输出字符串即可。

这里对于补充的编码长度如何写入呢？

程序优化：
可以对哈夫曼树进行压缩

参考资料:
[1].http://blog.csdn.net/u013275340/article/details/38778497?utm_source=tuicool&utm_medium=referral
[2].http://zzh87615.blog.163.com/blog/static/1178207282009516271866

```
#include <cstdio>
#include <cstdlib>
#include <cstring>

#define MAX 99999999

typedef struct node{
    unsigned char data;
    int weight;
    int parent,lchild,rchild;
    //unsigned char *bit;
    unsigned char *bit;
}HuffmanNode,HuffmanTree[512];
/*
typedef struct bit{//设立位域，用于解码进行数字比对
    unsigned a:1;
    unsigned b:6;
    unsigned c:1;
}Bit;
*/

void Select(HuffmanTree HT,int k,int &s1,int &s2){//选取最小的两个未链入哈弗曼树中的节点
    int minHuf1 = MAX,minHuf2 = MAX;
    s1 = 99999999,s2 = 99999999;
    for(int i = 1;i <= k;i++){
        if(HT[i].weight < minHuf1 && HT[i].parent == 0){
            minHuf2 = minHuf1;
            s2 = s1;
            minHuf1 = HT[i].weight;
            s1 = i;
        }else if(HT[i].weight < minHuf2 && HT[i].parent == 0){
            s2 = i;
            minHuf2 = HT[i].weight;
        }
    }
}

void Encode(HuffmanTree HT,int nReal){//进行字符编码
    unsigned char *cd;
    cd = (unsigned char*)malloc(nReal*sizeof(unsigned char));
    cd[nReal-1] = '\0';
    for(int i = 1;i <= nReal;i++){
        int start = nReal-1;
        for(int curr = i,parent = HT[i].parent;parent != 0 && 0 < parent && parent <= 2*nReal-1;curr = parent,parent = HT[parent].parent){
            if(curr == HT[parent].lchild)
                cd[--start] = '0';
            else
                cd[--start] = '1';
        }
        HT[i].bit = (unsigned char*)malloc((nReal-start+1)*sizeof(unsigned char));
        int x = 0;
        do{
            HT[i].bit[x++] = cd[start++];
        }while(cd[start-1] != '\0');
        HT[i].bit[x] = '\0';
        //strcpy(HT[i].bit,&cd[start]);
    }
    free(cd);
    /*
    if(nReal == 1)
        HT[1].bit[0] = '0';
        */
}

void Init(FILE *fpR,HuffmanTree &HT,int &nReal,int &n){//初始化哈夫曼树
    nReal = 0;
    HuffmanNode *p;
    int i;
    unsigned char ch;

    for(p = HT,i = 1;i <= 511;i++,p++)
        *p = {'\0',0,0,0,0};
    int start = 1;
    while(true){
        fread(&ch,1,1,fpR);
        if(feof(fpR))
            return;
        int tag = 0;
        n++;
        for(int j = 1;j <= nReal;j++){
            if(ch == HT[j].data){
                HT[j].weight++;
                tag = 1;
                break;
            }
        }
        if(tag == 0){
            HT[start].data = ch;
            HT[start++].weight++;
            nReal++;
        }
    }
}

void BuildTree(HuffmanTree &HT,int nReal){//建树
    int m = 2*nReal-1;
    for(int i = nReal+1;i <= m;i++){
        int s1,s2;
        Select(HT,i-1,s1,s2);
        HT[s1].parent = i;
        HT[s2].parent = i;
        HT[i].lchild = s1;
        HT[i].rchild = s2;
        HT[i].weight = HT[s1].weight + HT[s2].weight;
    }
}

void Decode(HuffmanTree &HT,int &n,int &nReal,FILE *fpR,FILE *fpW){//解码
    int sup;
    fread(&sup,sizeof(int),1,fpR);

    fread(&n,sizeof(int),1,fpR);
    fread(&nReal,sizeof(int),1,fpR);
    int i;
    HuffmanNode *p;
    for(p = HT,i = 1;i <= 511;i++,p++)
        *p = {'\0',0,0,0,0};

    for(i = 1;i <= 2*nReal-1;i++){
        fread(&HT[i].data,sizeof(unsigned char),1,fpR);
        fread(&HT[i].weight,sizeof(int),1,fpR);
    }

    BuildTree(HT,nReal);

    unsigned char c;
    unsigned char str;
    int root = 2*nReal-1;
    //i = 0;
    int length = 0;//对字符读取数量的计数，判断是否已经结束一次字符串读取
    int flagSup = 0;
    while(true){
        fread(&c,1,1,fpR);
        if(feof(fpR))
            return;
        //fseek(fpR,len,SEEK_CUR);
        for(int k = 0;k < 8;k++){
            if(HT[root].lchild == 0 && HT[root].rchild == 0){//哈弗曼树不存在度为1的节点，故用and或者or均可
                //if(HT[root].data == '\0')
                  //  fprintf(stderr,"Error str\n");
                str = HT[root].data;
                root = 2*nReal-1;
                fwrite(&str,1,1,fpW);
                length++;
            }
            if(length == n)
                break;
            if((c&128) == 0)
                root = HT[root].lchild;
            else if((c&128) == 128){
                root = HT[root].rchild;
            }
            c = c << 1;
            if(length == n)
                flagSup++;
        }
    }
}

void EncodeOutput(HuffmanTree HT,FILE *fpR, FILE *fpW,int n,int nReal){//输出编码
    fseek(fpW,4,SEEK_SET);
    fwrite(&n,sizeof(int),1,fpW);
    fwrite(&nReal,sizeof(int),1,fpW);
    for(int i = 1;i <= 2*nReal-1;i++){
        fwrite(&HT[i].data,sizeof(unsigned char),1,fpW);
        fwrite(&HT[i].weight,sizeof(int),1,fpW);
    }

    int length = 0;
    unsigned char c = 0;//编码变量
    unsigned char ch;//输入字符变量，作为文件顺序参照
    int sup = 0;
    int start = 0;
    fseek(fpR,0,SEEK_SET);
    for(int i = 0;i < n;i++){//进行逐个字符编码写入
        fread(&ch,1,1,fpR);
        for(int j = 1;j <= nReal;j++){//进行逐个匹配，寻找对应字符哈弗曼节点
            if(HT[j].data == ch){
                int StrLen = 0;
                while(HT[j].bit[StrLen] != '\0')//计算该节点字符的编码比特数
                    StrLen++;
                //fprintf(fpW,"%s",HT[j].bit);
                for(int k = 0;k < StrLen;k++){
                    if(HT[j].bit[k] == '0')
                        c = c << 1;
                    else if(HT[j].bit[k] == '1')
                        c = (c << 1)|1;
                    else fprintf(stderr,"Bit output error!\n");
                    length++;
                    if(i == n-1 && length % 8 != 0){//最后一个节点需要记录补充编码数
                        while(length%8!=0){
                            c = c << 1;
                            length++;
                            sup++;
                        }
                    }
                    if(length % 8 == 0 && length > 0)
                        fwrite(&c,1,1,fpW);
                }
                //fwrite(HT[i].bit,1,sizeof(HT[i].bit),fpW);
            }
        }
    }
    fseek(fpW,0,SEEK_SET);
    //fwrite(&length,sizeof(int),1,fpW);
    fwrite(&sup,sizeof(int),1,fpW);
    //fwrite(str,sizeof(unsigned char),length,fpW);
}

void InterfaceFile(char *infile,char *outfile){
    fprintf(stdout,"***Welcome to the Huffman Encoding/Decoding System:***\n");
    fprintf(stdout,"***                                                ***\n");
    fprintf(stdout,"Please input the input file name(including the postfix):");
    gets(infile);
    fprintf(stdout,"Please input the output file name(including the postfix):");
    gets(outfile);
}

void InterfaceNum(int &num){
    fprintf(stdout,"***Please choose which operation you'd like to do:****\n");
    fprintf(stdout,"***                                                ***\n");
    fprintf(stdout,"***1_Encoding          2_Decoding                  ***\n");
    while(fscanf(stdin,"%d",&num) && (num != 1 && num != 2)){
        fprintf(stdout,"The number is out of range.\nPlease input again:");
    }
}

int main(){

    FILE *fpR,*fpW;
    char infile[255],outfile[255];
    InterfaceFile(infile,outfile);
    if((fpR = fopen(infile,"ab+")) == NULL){
        fprintf(stderr,"File_Read open error!\n");
        exit(1);
    }
    if((fpW = fopen(outfile,"wb+")) == NULL){
        fprintf(stderr,"File_Write open error!\n");
        exit(2);
    }

    HuffmanTree HT;
    //unsigned char ch;//编码字符
    int n = 0,nReal = 0;//文件中字符数量n，哈弗曼节点数量nReal

    int num = 0;//选择方案
    InterfaceNum(num);
    switch(num){
        case 1:
            Init(fpR,HT,nReal,n);
            BuildTree(HT,nReal);
            Encode(HT,nReal);
            EncodeOutput(HT,fpR,fpW,n,nReal);
            break;
        case 2:
            Decode(HT,n,nReal,fpR,fpW);
            break;
    }

    fclose(fpR);
    fclose(fpW);
    return 0;
}

```