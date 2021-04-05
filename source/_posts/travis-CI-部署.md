---
title: travis CI 部署
date: 2018-03-19 22:39:02
updated:
tags: [automate, travis]
categories: [专业]
---

根据参考1部署了一下自动化travis，

然后自动化部署依旧报错，终于意识到了next主题无法正常运作的原因，是配置文件过时了的原因，没跟上next版本迭代的速度，所以其中有些配置信息的导入造成了错误。

重新修改之后，终于可以在travis CI正常运作了。

在更改为pandoc解析引擎时，忘记了hexo-renderer-pandoc是依赖于pandoc来执行的，按照[2]在travis.yml里添加了安装pandoc环境后就可以正常运行了。

# 参考:
1. [用 Travis CI 自动部署 hexo segmentfault](https://segmentfault.com/a/1190000004667156)
2. [travis CI pandoc](https://peterxugo.github.io/2017/05/27/hexo%E5%86%99%E5%8D%9A%E5%AE%A2/)
