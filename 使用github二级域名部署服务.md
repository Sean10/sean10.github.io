---
title: 使用github二级域名部署服务
date: 2018-04-13 21:42:36
updated:
tags: [github, DNS]
categories: [专业]
---

以前比较偏向买个域名，然后理论上用服务商提供的dns解析根据不同的sub domain指向不同ip就能部署服务了。其中，将github page CNAME到自己的域名上。

如果是使用提供的二级域名作为主域名呢？域名指向权限不在自己手中的情况下，github如何提供的解决方法呢？

<!--more-->

github支持gh-pages分支生成项目页`<username>.github.io/<project>`，但还是不能利用自己的服务器部署服务。

在参考1中提出了一个利用gh-pages+HTML 302的方法进行跳转的方法。

再仔细找了找，除了自定义域名和这个302的方法，Github官方没有开放这个域名指向其他服务器的权限，仔细想想这样也是有道理的。否则岂不是github的域名也无法保证这个网页的安全性了。

# Reference
1. [把 Github 用作 DNS 设置二级域名跳转](https://drkbl.com/use-github-as-dns/)
