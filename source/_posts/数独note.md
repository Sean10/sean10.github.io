---
title: 数独note
date: 2018-02-24 19:40:46
updated: 2018-02-27 17:00:00
tags: [数独]
categories: [专业]
---

unique rectangle type 1~4没什么疑问，不过hidden unique rectangle 的删除有些疑惑，如何确定哪个是候选数a，哪个是候选数b的呢

<!--more-->

有些技巧似乎国内的那个论坛上也并没有提到，国外那位大神总结的真是全。

## Generalising X-Wing

>X-Wing is not restricted to rows and columns. We can also extend the idea to boxes as well.
If we generalise the rule above we get:
>
>When there are
> * only 2 candidates for a value, in each of 2 different units of the same kind,
> * and these candidates lie also on 2 other units of the same kind,
>
>then all other candidates for that value can be eliminated from the latter two units.
>
>Now we have 6 combinations:
>
> * Starting from 2 rows and eliminating in 2 columns   -> Classic X-Wing
> * Starting from 2 columns and eliminating in 2 rows   -> Classic X-Wing
> * Starting from 2 boxes and eliminating in 2 rows	    -> Same effect as line/box reduction
> * Starting from 2 boxes and eliminating in 2 columns  -> Same effect as line/box reduction
> * Starting from 2 rows and eliminating in 2 boxes     -> Same effect as pointing pairs
> * Starting from 2 columns and eliminating in 2 boxes  -> Same effect as pointing pairs



# 参考资料：
1. [SudokuWiki](http://www.sudokuwiki.org)
