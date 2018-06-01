---
title: zotero使用记录
date: 2018-05-16 14:52:06
updated:
tags: [zotero, jabref]
categories: [专业]
---

为了管理论文文献，方便引用导出等等，要用zotero,但是单纯这个导出的bibtex又无法直接给xelatex使用，就试着使用一下jabref这个工具

<!--more-->
# 问题集锦

## xelatex导出bib参考文献，格式不正确，输出有些混乱[2]

使用zotero导出的bib中，文献类型同样是学位论文，但是bib中是phdThesis,而我本科的模板上是MasterThesis,暂时没找到如何让zotero修改这个的方法，倒是找到了在JabRef上修正这个的方法，看来暂时只能用JabRef了.

不过在LaTex的bst格式文件中也找到了phdthesis的输出样式，所以错误原因究竟是什么呢？

经过一个个与模板bib文件比照，发现是由于缺少了`language={Chinese}`这行设置，补充后即可输出了，搞定。

由于JabRef的无法正常运行，又去了解了下zotero的强大，经过检索，发现还是zotero也是可以自己配置一些可选field之后导出bib的，在language位置设置为chinese，搞定。

## Zotero中文文献作者只显示姓[1]

根据知乎上作者所说，是在抓取网页解析的语法出现了问题，所以按照知乎所说修改解析方法，全部防止到姓中即可
>1. 首选项-高级-文件和文件夹-打开数据文件夹。
>找到translators文件夹，找到里面的CNKI.js，打开
>2. 注释掉189和190两行，在后面加上一行 creator.firstName = "";
>3. 刷新文献所在页面，重新获取条目，创建者即为作者全名。
>4. 文件重命名规则可使用%a

## JabRef 在打开finder时，经常进入 not responding状态

按照issue里说的换成4.3 beta版本就可以正常使用了,唔，在新建entry的时候还是和issue一样Freeze了。

# Reference
1. [Zotero 作者按西文的方式显示，如何调整为中式？](https://www.zhihu.com/question/58599043)
2. [参考文献管理工具zotero的使用经验分享](http://muchong.com/html/201410/7981977.html)
