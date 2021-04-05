---
title: LaTex学习记录
date: 2018-05-04 20:30:59
updated:
tags: [LaTex]
categories: [专业]
---

<!--more-->


# 编译过程

``` latex
xelatex main.tex
bibtex main.aux
xelatex main.tex
xelatex main.tex
open -a preview main.pdf
```

将tex转成word
```
pandoc -s main.tex -o main.docx
```


```
# 调用方法 ：
# 单个文件
texcount  your-file-name.tex
# 多个文件
texcount file-name1.tex file-name2.tex
```

# 出现问题

>---they aren't the same literal types for entry shenyaoZhongGuoDianYingZaiXianPiaoWuFaZhanYanJiu2016
while executing---line 2571 of file buptbachelor.bst
0 is an integer literal, not a function, for entry shenyaoZhongGuoDianYingZaiXianPiaoWuFaZhanYanJiu2016
while executing---line 2571 of file buptbachelor.bst
You can't pop an empty literal stack for entry shenyaoZhongGuoDianYingZaiXianPiaoWuFaZhanYanJiu2016
while executing---line 2571 of file buptbachelor.bst

根据检索，按照4次编译过程来[4]，就可以得到文献结果，只不过出现中文编码错误


>英文参考文献中出现中文字符

对language域进行判断的话，需要改bst文件，针对一些特殊的域进行判断，比较麻烦。
现成的GBT7714的那个，只要language域不是空的，就按中文输出。[5]




# Reference

1. [Latex中bib参考文献的编译](https://blog.csdn.net/u014466412/article/details/53282615)
2. [bib格式](http://blog.sina.com.cn/s/blog_4aee288a0100dehr.html)
3. [使用 BibTeX 生成参考文献列表](https://liam0205.me/2016/01/23/using-bibtex-to-generate-reference/)
4. [Latex下使用IEEEtran模板编译bib失败报错的解决方法](https://blog.csdn.net/greenapple_shan/article/details/37054813)
5. [对于中英文参考文献，如何区别输出？](http://bbs.ctex.org/viewthread.php?tid=33591)
