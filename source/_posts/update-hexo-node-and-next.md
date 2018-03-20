---
title: update hexo node9 and theme next
date: 2018-02-17 02:14:16
updated: 2018-02-24 16:00:00
tags: [hexo, node]
categories: [专业]
---

之前没有用nvm管理node版本，导致node升级到9.5以后，hexo里的依赖都不能正常运作了，单独一个个升级依赖没能成功解决bug.

<!--more-->

在hexo里已经支持了update时间，在模板里添加updated即可，在next 主题配置里，修改post_meta为true即可

**参考**
[front-matter](https://hexo.io/zh-cn/docs/front-matter.html)

-------

```
Error: write EPIPE
at _errnoException (util.js)

at WriteWrap.afterWrite [as oncomplete] (net.js)
```
遇到这个错误，重装好久，也没解决，在[#2913](https://github.com/hexojs/hexo/issues/2913)里找到，更新了hexo-renderer-pandoc就no error了。

-----

中途，在一个个卸载plugin判断错误来源，发现卸载了hexo-renderer-stylus之后，就无法生成css样式了。

-----


遭遇白屏问题，似乎字号颜色渲染问题？只有下方的copyright可见，内容虽然可以点击，但是不可见，全选也看不到文字颜色反转（所以其实也不算字体颜色问题？）

似乎这是一个next久远的bug.

在network看到是fancybox.js的加载404，在console里看到的是config is not defined, next is not defined，导致后续无法识别.

根据[这条issue](https://github.com/iissnan/hexo-theme-next/issues/1896)关闭了motion就可以正常显示了。

-------

No layout问题。

为了升级next，发现_config.yml是一个非常影响的文件，官方说hexo3支持了data file功能，可以将站点和主题config都放到到_data/next.yml中，在翻译的文档中都没有提到一点，原始的站点配置不能删除

>Then, in main site's hexo/_config.yml need to define theme: next option (and if needed, source_dir: source).

[来源](https://github.com/theme-next/hexo-theme-next/blob/master/docs/DATA-FILES.md)

弄了好久……，恢复站点配置文件之后，data-file就生效了

----------

关于pandoc解析mathjax，目前还是存在错误，view状态也存在错误

需要在每篇需要的md Frontter开启mathjax: true

---------

首页不显示最新的，以及分类页面缓存了之前的错误
