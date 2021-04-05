---
title: hexo开启atom订阅
subtitle: hexo
date: 2020-06-27 22:45:43
updated:
tags: [hexo]
categories: [专业]
---

最近想起来, 收藏的博客都没怎么看, 最近全靠公众号在提供一些有意思的内容,但是这个渠道还是太单薄了.

想起来之前用`inoreader`订阅过一些频道, 现在把最近一段时间收藏的博客再给补充进去, 顺便把自己博客的`atom`源也给打开. 

PS. 扫了一遍,好多用自定义域名的博客都已经不再维护, 这些不怎么常见的域名像是被专门去买别人未续费的域名机构给收集了,跳转到了一些做`SEO`\域名交易联系的网站之类的. 还是github这种基本有几乎永久保障的二级域名好,.

``` 
# npm install hexo-generator-feed --save

feed:
  type: atom
  path: atom.xml
  limit: 0
```

在打开生成的atom.xml时输出这样一个错误`PCDATA invalid Char value 8`, 最后发现是`md`文件中多出了`^H`这个符号, `vs code`暂时不展示这种字符. 暂时也不知道是在什么情况下被添加进来的. 总之, 按照[^1]通过`vim`找到并删除之后就可以了.

## vs code render
按照有些大佬说的,设置`Render Whitespace`成`all`可以看到一些字符了,但是这个里还是只显示`space`和`tab`,似乎并不能显示换行符及像上面这种`^H`符号. 

### 遇到问题

本来想着常开这个,至少能起到看我有没有在某些文件里错用tab,发现这个功能在markdown文件编写标题时,会出现和输入法重复的问题, 会出现下述这种效果.

![](hexo开启atom订阅/hexo开启atom订阅_2020-06-28-18-04-35.png)

## Refernence
1. [Hexo 的 RSS 生成错误 \- ouyangsong \- 博客园](https://www.cnblogs.com/ouyangsong/p/9348169.html)