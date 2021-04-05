---
title: vscode相关技巧
subtitle: tech
date: 2021-01-31 16:32:59
updated:
tags: [vscode]
categories: [专业]
---

# todo
## 目前vscode似乎还没有支持一个当前文件所有搜索项的列表显示的功能
* 暂时是通过左侧的全局搜索, 定位到当前文件来进行搜索的.


## 类似typora的所见即所得插件支持
* 即时渲染
* 有一些unote之类的`web editor`嵌入到`vscode`中打开, 这样的话也算是实现了一个所见即所得的编辑器, 不过不是基于Vscode的倒是了. 不过这种情况下有些vscode配置的快捷键应该还是可以用的.



## 同义词模糊搜索功能(fuzzy search)
* fzf
  * fuzzy search
  * 只能搜索文件名, 还是有点欠缺的
* 不知道这个模糊搜索能不能做到NLP的水平呢?
* 中文的似乎暂时没有, 别的都是直接基于`fzf`的

## 是不是有什么功能能够搜索整个目录里所有的todo内容呢?


## vim插件使用
### vim覆盖了vscode的`CMD`和`Ctrl`的快捷键
> 可以通过 File -> Preference -> Settings中   vim.useCtrlKeys 选项设置为 false

### vim无法使用系统剪切板
打开设置中的vim的Use System Clipboard


# 窗口之间的焦点切换

> ctrl + M 使用tab键切换焦点模式tab move foucs
> ctrl + ~ 聚焦到TERMINAL(展开/收起TERMINAL)
> ctrl + 聚焦到editor
> Ctrl + Shift + E 资源视图和编辑视图的焦点切换
> Ctrl + Shift + V 预览Markdown文件【编译后】
> Ctrl + K v 在边栏打开渲染后的视图【新建】



# vim
## neovim
据说最多人用的`vim`插件冲突太多, 但是`neovim`就会好转很多.

更换之后, 至少`Ctrl+c`是没有问题的, 在`insert`模式下

## vim插件快速打开关闭
```
Command->toggleVim
```
# vscode 插件资源占用排查
* 打开`Help->Process Explorer`,  这里会显示出每个窗口的`CPU`和内存的占用.


# 中文输入markdown时, 需要2次backspace才能删除掉在输入法中的字符
编辑器不会跟随同步删除.

根据[^2]来看,似乎是`electron`和`chromium`的问题. 但是我用的`1.51`的`vscode`还是存在这样的问题呀?为什么关闭了呢?

似乎我升级了下搜狗输入法就好了, 至少5.9.0.11685目前没什么问题.

# 快捷键
快速切换tab, `Option+Cmd+左右方向键`


# 中文分词[^4]
因为最近写文字记录比较多, 有时候词写错了, 想快点删的时候只能一个个删, 不像因为单词可以直接删, 就忽然意识到. 现在分词做的这么好了, 理论上这种插件应该已经有了.

果然, 搜到了这个[CJK Word Handler \- Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SharzyL.cjk-word-handler), 试了下, 还挺好用.

# markdown的`Cmd+B`覆盖了`vscode`的`outline`打开和关闭. 
* 被markdown覆盖掉了怎么办? 我暂时选择删除markdown的这个快捷键



# Reference
1. [\(2\) vscode控制字符引起的问题以及解决思路\_洞香春 \- SegmentFault 思否](https://segmentfault.com/a/1190000013357949)
2. [Backspace can not erase the last one character during Chinese/Japanese IME conversion \(macOS\) · Issue \#24981 · microsoft/vscode](https://github.com/microsoft/vscode/issues/24981)
3. [Backspace can not erase the last one character during Japanese IME conversion \(macOS\) · Issue \#9173 · electron/electron](https://github.com/electron/electron/issues/9173)
4. [VSCode 中文分词插件：CJK Word Handler \- 知乎](https://zhuanlan.zhihu.com/p/148195204)