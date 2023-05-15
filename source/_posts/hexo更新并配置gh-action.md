---
title: hexo更新并配置gh_action
subtitle: tech
date: 2023-03-18 00:18:47
updated:
tags: [hexo]
categories: [专业]
---


# 本地适配最新版hexo

# 配合action操作

修改action
```

git push origin --tags
```

或者直接找相对路径

[Workflow syntax for GitHub Actions \- GitHub Docs](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-an-action-in-the-same-repository-as-the-workflow)

处理成功之后, 相对还是比较简单的?


### 当前版本包
``` json

"hexo": "^4.2.1",
"hexo-asset-image": "^1.0.0",
"hexo-deployer-git": "^0.3.1",
"hexo-douban": "^0.2.9",
"hexo-generator-archive": "^0.1.5",
"hexo-generator-category": "^0.1.3",
"hexo-generator-feed": "^2.2.0",
"hexo-generator-index": "^0.2.1",
"hexo-generator-json-content": "^3.0.1
"hexo-generator-tag": "^0.2.0",
"hexo-inject": "^1.0.0",
"hexo-migrator-rss": "^0.1.2",
"hexo-renderer-ejs": "^0.3.1",
"hexo-renderer-less": "^1.0.0",
"hexo-renderer-pandoc": "^0.2.2",
"hexo-renderer-stylus": "^0.3.3",
"hexo-server": "^0.3.1",
"hexo-symbols-count-time": "^0.3.2",
"hexo-util": "^1.9.1"

```

以前引入了这些组件, 待会升级时需要处理


### 更新插件

``` bash
npm install -g npm-upgrade
npm-upgrade
npm udpate --save
```


### `_config.yml`更新


##### `external_link` deprecated

更改格式为以下

```

external_link: 
  enable: true
  field: site
  exclude: []

```

##### 更新theme版本

比如`indigo`更换[scarqin/hexo\-theme\-indigo: 维护 hexo\-theme\-indigo 自用](https://github.com/scarqin/hexo-theme-indigo/tree/hexo6)


##### style.css 渲染失败 

```
ERROR Asset render failed: css/style.css
Unrecognised input
```

发现style.css是空的

说明less生成失败了

折腾了半天, Valine.less文件有问题, 更新了上面repo里的文件就恢复正常了


##### 图片链接都变成`.io`前缀的


~~根据该issue可知[Package hexo\-asset\-image doesn't return correct image path · Issue \#4930 · hexojs/hexo](https://github.com/hexojs/hexo/issues/4930), 是用了过时的`hexo-asset-image`引起.卸载即可.~~

通过[Hexo: wrong address for meta and img tags \- Stack Overflow](https://stackoverflow.com/questions/62512085/hexo-wrong-address-for-meta-and-img-tags) 更换了另一个插件[cocowool/hexo\-image\-link: 当MD中引用本地文件时，处理生成的html中的图片链接。](https://github.com/cocowool/hexo-image-link)解决的.
