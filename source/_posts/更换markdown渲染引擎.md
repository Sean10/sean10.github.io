---
title: 原计划更换markdown渲染引擎
date: 2017-07-01 17:01:14
tags: [hexo, markdown]
categories: [随想]
---

前两天在写复习笔记的时候，发现渲染出的mathjax公式不像本地预览时显示的那样正常，含有`_`符号的都被渲染成了`<em>`标记，导致公式显示异常。网上推荐更换渲染引擎hexo-renderer-krame，虽然这个是fork出来特地解决了hexo-renderer-marked的这个问题的，但是同样，也修改了markdown的一些基本语法，在写行内公式时，就需要在$$外套上一层\`符号，而我本地的预览使用的渲染引擎同样也会需要修改，这就比较麻烦了。

再进行了搜索，发现了一个据说非常强大的引擎pandoc，不过我在安装后，始终遇到以下错误。

>INFO  Start processing
>FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
>
>Error: [pandoc warning] YAML header is not an object "source" (line 67, column 1)\
>    at ChildProcess.<anonymous> (/Users/sean10/Code/sean10.github.io/node_modules/hexo-renderer-pandoc/index.js:73:20)\
>    at emitTwo (events.js:106:13)\
>    at ChildProcess.emit (events.js:191:7)\
>    at maybeClose (internal/child_process.js:877:16)\
>	 at Process.ChildProcess._handle.onexit (internal/child_process.js:226:5)

经过搜查，这似乎属于pandoc解析时的错误，找到了以下类似内容。


>See http://johnmacfarlane.net/pandoc/README.html#yaml-metadata-block
>
>There must be something in your document that looks like a YAML metadata block, but isn't. Such a block would start with---on a line by itself and end with---or...on a line by itself. The line numbers in the error message refer to lines inside the metadata block, not to lines of the document.\
>By the way, you can turn off YAML metadata block parsing entirely by putting
>```
--from markdown-yaml_metadata_block
```
>in your pandoc command line.

不过这个解决方法并没能解决问题，我换了一个思路。

后来想了想，是否可能是在渲染文章头部YAML格式时出现错误，导致后续的markdown就无法渲染了，所以可能是在文章内容上出现了问题，但是这个error信息没能显示具体文件，又没有日志，我决定二分排查是哪篇的错误。

很快就发现了，是最近写的Unix复习那篇的错误，在第67行，没有意识到错误所在，经过测试，发现问题是在于markdown的语法上，pandoc语法要求标题#前必须插入一空行，修改后终于成功渲染了。真是不容易。

>
>Extension: blank_before_header
>
>始 markdown 語法在標題之前並不需要預留空白行。Pandoc 則需要（除非標題位於文件最開始的地方）。這是因為以 # 符號開頭的情況在一般文字段落中相當常見，這會導致非預期的標題。例如下面的例子：
>
>I like several of their flavors of ice cream:
>```
#22, for example, and #5.
```

# 参考文献
1. [parse yaml](https://stackoverflow.com/questions/19520463/pandoc-cannot-parse-yaml-header-when-converting-md-to-pdf)
