---
title: vscode之markdown引用zotero参考文献
tags:
  - zotero
  - vscode
categories:
  - 专业
subtitle: tech
date: 2021-01-17 16:40:53
updated:
---



# 需求

我的主要需求, 只是当我需要写引用时, 排序能够自动调整,而不是我手动去填写数字. 如果能够让最后的参考文献也能够像用`Latex`时一样自动生成就更好不过了.

初步的一个思路是, 因为我主要需要的是纯文本, 然后在使用`hexo-render-pandoc`生成为网页时, 能够把索引给使用上. 预计是在使用`pandoc`的生成过程中, 将`markdown`文件中的引用的`key`给转换成对应的索引.

目前根据[Sci\.Fun \| 如何使用markdown撰写论文？ \- 知乎](https://zhuanlan.zhihu.com/p/103234043)这些来看, 跟使用`pandoc`时导入`bib`的`library`基本一致.

搜集一下有没有现成的.

暂时好像是没找到这个结论.

## 一个简易方案[^sspai][^pandoc]

[^pandoc]: [Pandoc \- Pandoc User’s Guide](https://pandoc.org/MANUAL.html#footnotes)

[^sspai]: [5\.4 如何用 Markdown 写论文？ \- 少数派](https://sspai.com/post/57082)


在引用中使用[^vscode-zotero], 在参考文献位置则是即可完成链接的功能.

[^vscode-zotero]: [mblode/vscode\-zotero: Zotero Better Bibtex citations for VS Code](https://github.com/mblode/vscode-zotero)

```
[^1]:xxx
```

更进阶的方案应该需要看使用的`render`, 比如我的是`pandoc`,就看`pandoc`的引用相关的文档即可.

[Pandoc \- Pandoc User’s Guide](https://pandoc.org/MANUAL.html#pandocs-markdown)
Citation rendering

### 问题
#### 前面的引用的后面的链接的脚注编号变成从1开始

所以脚注内的内容, 最好不用编号, 而是实际的指代. 且将参考文献直接放在该段后. 通过渲染自动挪到文章/页最后.
