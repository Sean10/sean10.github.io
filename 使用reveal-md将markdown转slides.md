---
title: 使用reveal-md将markdown转slides
subtitle: tech
date: 2020-06-21 16:57:16
updated:
tags: [工具, node-js, markdown, pdf]
categories: [专业]
---

前段时间看到了`vue-mark-display`这套功能，直接基于markdown生成slides，都不用去专门学习beamer那套，直接就能基于现有的markdown生成，感觉相当不错。

不过在折腾这套工具的时候，遇到一个问题，自己展示的时候用html没有问题，但是归档发布给其他人的时候，当然最好还是以pdf的形式嘛。这套工具不知道是以为没适配高分屏还是什么原因，通过Chrome和decktype导出pdf时，文字布局一定会与页面上看到的不一致。

具体也没细看了，这套工具终究是个人使用为主，可能开发者的使用场景不怎么需要导出，比如纯外网或内网部署了自己的静态网站之类的情况下。

考虑到这套工具偏个性化，于是去找找适用性较广，功能做的比较完善的这类产品。

发现了`reveal.js`这套目前来看star数应该是最多的，也支持所有我需要的功能，如markdown\vue支持\pdf导出等等。还有人基于这套，做了一个一键使用的`reveal-md`，更加满足我的需求了。

这篇文章主要讲的就是针对我的几个痛点，我怎么使用这套工具了。

emm，发现几个问题，看看remark和remark-slide和spectacle这俩star更多的有没有解决这个问题呢？`reveal-md`作者没兴趣修这些问题……

## 指定主题

``` bash
# 指定官方提供的
reveal-md slides.md --theme solarized
# 自定义scss
reveal-md slides.md --theme theme/my-custom.css
```

## 文件内嵌yaml输出参数

``` markdown
---
title: temp
theme: solarized
revealOptions:
    transition: 'fade'
---

## slide 1

slide 

---

##  slide 2

这种效果

```

## 输出pdf（不建议使用，不支持mathjax）

``` bash
reveal-md slides.md --print slides.pdf
```

html看到的公式和图片都是正常的。但是基于`Puppeteer`导出的时候，公式无法加载。按照`reveal.js`里说的，是已经支持了部分版本的mathjax了的，所以只能从这个工具使用的导出方式上来怀疑了。

可能因为headless浏览器那边加载的问题？没加载指定的js之类的？导致无法解析出`mathjax`公式。可能是这个`reveal-md`里对puppeteer的使用上存在一些问题？因为看decktape底层应该也是基于puppeteer来做的。



## 使用reveal.js的print-pdf（推荐使用）

~~emm，有人说reveal.js原生的`?print-pdf`已经修复了这些问题，但是reveal-md，怎么像是css不加载？~~

要在html后的`#`前增加?print-pdf，在`#`后面加是没用的……总算是意识到了

```
http://localhost:8000/index.html?print-pdf#/
```

## 通过Decktape输出pdf(支持mathjax，但暂时存在标题与图片间隙过大问题未修复，pinned状态)

使用`reveal-md`提供的生成Static文件的方式，再通过`decktape`将静态网页转成pdf。

``` bash
reveal-md temp.md --static static
decktape index.html new.pdf
```

根据下面[^2]里提到的问题，目前找到的有两种解决方式
1. 使用screenshots输出，再写个脚本拼接成pdf，可能会丢失一些动态效果？如果尺寸比图片小，也会丢失下面的内容……不过这个和html是表现一致的，所以如果图片在html有问题，还可以调整。
2. 使用增加size的方式，直到找到可以正确显示的方式。但是这样的size针对不同的图片大小，还需要手动调整……
3. 使用1.0.0版本的decktape，这个版本还在使用phantomjs，但是我发现在高分屏幕环境，似乎存在一点问题.只能显示一部分，原本的中间的内容跑到了右下角去了。
4. 使用原生的print-pdf，但是好像我这边生成的好像怎么都加载不了css，位置始终在左侧.

常用尺寸：
* 2048x1536
* 1920x1080
* 2560x1440(screenshots)



### 存在图片和标题间距过大问题[^2]

绕过方式如下:

>A workaround seems to be increasing (very much) the size. For example using --size='2048x1536' instead of --size='1024x768' works for me.

但是这个存在一个问题，因为图片分辨率一旦超过设置的宽度，他这边就会无法显示了。

可能相比之下还不如刚才那样直接输出。如果使用screenshots输出倒是没有一点问题。


## 备注

在markdown文件每页中增加`Note:`，这页的剩下的部分就都会显示在注释中，打开本地web端访问slides后，按`s`会弹出注释页面，放到其他屏幕上就可以了。然后分别按`Cmd+Ctrl+f`全屏即可了。[^5]

只不过这种只适合多屏幕场景了吧，不过对于投屏来说，的确就是多屏幕

## Reference
1. [webpro/reveal\-md: reveal\.js on steroids\! Get beautiful reveal\.js presentations from any Markdown file](https://github.com/webpro/reveal-md#theme)
2. [weird spacing with reveal · Issue \#151 · astefanutti/decktape](https://github.com/astefanutti/decktape/issues/151)
3. [astefanutti/decktape: PDF exporter for HTML presentations](https://github.com/astefanutti/decktape)
4. [remark vs remark\-slide vs reveal\-md vs reveal\.js vs spectacle \| npm trends](https://www.npmtrends.com/remark-vs-remark-slide-vs-reveal-md-vs-reveal.js-vs-spectacle)
5. [Presenter mode · Issue \#404 · hakimel/reveal\.js](https://github.com/hakimel/reveal.js/issues/404)