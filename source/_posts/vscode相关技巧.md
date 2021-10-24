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

## 搜索窗口切换到下一个匹配对象

* F4
* shift+F4 回到上一个

[visual studio code \- vscode key binding for "goto next search result on the search results pane"? \- Stack Overflow](https://stackoverflow.com/questions/39773141/vscode-key-binding-for-goto-next-search-result-on-the-search-results-pane)

# vim
## neovim
据说最多人用的`vim`插件冲突太多, 但是`neovim`就会好转很多.

更换之后, 至少`Ctrl+c`是没有问题的, 在`insert`模式下

### 配置默认insertmode


``` vim
" 添加vim开启默认insert
autocmt WinEnter * startinsert
```

[Setting to open editors in insert mode by default · Issue \#613 · asvetliakov/vscode\-neovim](https://github.com/asvetliakov/vscode-neovim/issues/613)

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

# 所见即所得插件(markless)

哇, 今天偶然看到的这个插件, 太厉害了, 完全兼容我其他的插件的快捷键, 不需要学习成本, 直接就能享受`typora`的效果.

开发修改

# 关闭回车会触发自动补全, 修改为tab. 避免无需补充时, 不得不补充问题
在`Suggestions`里的`Accept Suggestion On Enter`. 修改为`Off`即可


# 快捷键
快速切换tab, `Option+Cmd+左右方向键`

通过`Cmd+1`等切换主窗口`cursor`.


# 中文分词[^4]
因为最近写文字记录比较多, 有时候词写错了, 想快点删的时候只能一个个删, 不像因为单词可以直接删, 就忽然意识到. 现在分词做的这么好了, 理论上这种插件应该已经有了.

果然, 搜到了这个[CJK Word Handler \- Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SharzyL.cjk-word-handler), 试了下, 还挺好用.

# markdown的`Cmd+B`覆盖了`vscode`的`outline`打开和关闭. 
* 被markdown覆盖掉了怎么办? 我暂时选择删除markdown的这个快捷键


# 自动补全
## TabNine AI自动补全

# git
## gitlens

# remote develop
remote ssh配置环境变量的时候

需要配置一下`~/.profile`, 因为`~/.bashrc`在非交互式窗口不会被初始化. 

# Intelligence
## python
一开始用的`pylance`和`jedi`. 这次看`pyx`代码的时候这俩都不支持, 就换回了`microsoft python`.


# highlight
`shift+f8`高亮, 应该类似`SourceInsight`的效果.

# C/C++索引

在`settings.json`中增加下述
```
"files.exclude": {
      "**/.xxxx": true
},
"search.exclude": {
    "**/.xxxx": true
 },

```

而不用在`c_cpp_properties.json`中增加任何处理.

但是最好创建个这个文件, 好像就能够索引了.

# container使用

[Developing inside a Container using Visual Studio Code Remote Development](https://code.visualstudio.com/docs/remote/containers)

[Get started with development Containers in Visual Studio Code](https://code.visualstudio.com/docs/remote/containers-tutorial)

## container版本就是在满足的linux开发容器里再装一个vscode-server

## remote版本, 就是单纯的自己选一个编译环境,  然后装好vscode-server的服务器上连接上.

## 插件安装失败, 指定代理地址.

[visual studio code \- VSCode Remote Container \- extensions not installing on dev container using docker\-compose \- Stack Overflow](https://stackoverflow.com/questions/55992660/vscode-remote-container-extensions-not-installing-on-dev-container-using-docke)
mac上的Docker是跑在虚拟机里的, 所以即便设置`--net=host`, 实际上还是和宿主机差一级, 所以这里指定宿主机的内网ip的话, 其实是可以通讯了的.


## vscode 卡顿, `process Explorer`中显示`extension Host` cpu 100%+

根据这个[Performance Issues · microsoft/vscode Wiki](https://github.com/Microsoft/vscode/wiki/Performance-Issues#profile-the-running-extensions)定位方式, 查到`profiling time`中`todo tree`插件占用了4s多, 可能是因为这个文件中内容太多, 导致慢了. 把这个插件禁用, 就不卡顿了.

# todo
## 存在一个问题, 不知道为什么有时候`cmd+c`和`cmd+v`在比如find或者`ctrl+p`的窗口无法粘贴.



# Reference
1. [\(2\) vscode控制字符引起的问题以及解决思路\_洞香春 \- SegmentFault 思否](https://segmentfault.com/a/1190000013357949)
2. [Backspace can not erase the last one character during Chinese/Japanese IME conversion \(macOS\) · Issue \#24981 · microsoft/vscode](https://github.com/microsoft/vscode/issues/24981)
3. [Backspace can not erase the last one character during Japanese IME conversion \(macOS\) · Issue \#9173 · electron/electron](https://github.com/electron/electron/issues/9173)
4. [VSCode 中文分词插件：CJK Word Handler \- 知乎](https://zhuanlan.zhihu.com/p/148195204)