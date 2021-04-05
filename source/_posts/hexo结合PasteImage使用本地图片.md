---
title: hexo结合PasteImage使用相对路径图片
subtitle: tech
date: 2020-06-24 00:46:35
updated:
tags: [hexo, vs code]
categories: [专业]
---

## 背景

最近发现用外部OSS图床，在本地处理归档文件资源时，不是那么的方便。且自己用的腾讯云COS的上传插件没做对剪切板中图片上传的功能，导致每次粘贴图片都得多一步去获取文件的绝对路径或者选中文件。

然后发现`vs code`有个`Paste Image`的插件可以做到直接将剪切板的图片放到本地指定相对路径的功能。

但是对于发布到`github page`上的文章呢，图片资源怎么才能使用相对路径呢？

就发现`hexo 3`提供了一部分功能（即便现在4了，好像也暂时没有更进一步优化的计划，relative path没怎么搜到有人提PR了）。

按照hexo的意思是不使用标准markdown语法，而是使用下面这种提供的外部语法来处理……

```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

不太能接受，然后发现配合一个好多年前的插件`hexo-asset-image`就可以做到对于用户可以无感知的使用相对路径的图片了。

## hexo配置
0. hexo版本大于3，node版本不知道有没有影响（我本地14.4版本无法正常工作，9.5可以）
1. _config.yaml中`post_assets_folder`设置为true
2. npm install hexo-asset-image
3. vim node_modules/hexo-asset-image/index.js 按照[^2]修改代码
   1. 原因在于作者不再维护了，npm源里一直没有重新打包的内容

``` diff
-  var endPos = link.lastIndexOf('.');
+  var endPos = link.length-1;
```

原始的`md`文件中的图片路径依旧可以用正常的相对路径`![](hexo结合PasteImage使用本地图片/hexo结合PasteImage使用本地图片_2020-06-24-00-47-39.png)`， 在`hexo g`时生成的文件中的图片资源路径会被这个插件自动转换成url后静态文件相对于静态文件根目录的绝对路径。

## PasteImage配置

``` json
{
    "pasteImage.namePrefix": "${currentFileNameWithoutExt}_",
    "pasteImage.defaultName": "YMMDDHHmmss",
    "pasteImage.path": "${currentFileDir}/${currentFileNameWithoutExt}",
    "pasteImage.basePath": "${currentFileDir}",
    "pasteImage.forceUnixStyleSeparator": true,
}
```

### 图示配置如下
![](hexo结合PasteImage使用本地图片/hexo结合PasteImage使用本地图片_2020-06-24-00-47-39.png)


## Reference
1. [Hexo开启post\_asset\_folder后, 安装hexo\-asset\-image,不起作用的问题 \| Tomtan's Blog](http://www.itomtan.com/2017/09/29/the-problem-when-use-post-asset-folder/)
2. [域名是xxx\.io的情况下，图片路径会从原本/xxx\.jpg变成 /\.io/xxx\.jpg · Issue \#47 · xcodebuild/hexo\-asset\-image](https://github.com/xcodebuild/hexo-asset-image/issues/47)