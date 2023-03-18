---
title: windows terminal1.0暂时还无法替代tmux
subtitle: terminal
date: 2020-05-25 13:57:05
updated:
tags: [windows, terminal, tmux]
categories: [专业]
---

前段时间, windows Terminal 1.0发布了，其实我知道这个的时候，还是非常期待的。毕竟公司的开发机是windows，需要一直连接着linux设备做开发。
<!--more-->

我之前的方案是自己搭了个本地虚拟机做跳板，这个虚拟机里装了tmux，然后我通过windows上可以ssh的工具ssh上去使用`tmux`， 再开多个`pane`ssh开发机。

tmux通过快捷键切换、创建、分割、临时全屏化pane来操作还是非常便捷的。

目前我是使用的putty作为ssh工具。之前使用过cygwin（MSYS)\Cmder\ConEmu等window上比较成熟的工具，不过在这些环境下启动tmux，当时是在创建切换pane上有些或多或少的问题。听说MSYS2之前也已经出了，似乎已经比较成熟了，大佬们也可以试试。

我最近已经比较图稳定了，之前遇到的或多或少的问题有比如我需要显示编码从UTF-8临时切换到GBK，在这个时候有些工具会把分隔栏变成错误字符。还有我需要简洁的全屏，只需要通过tmux来控制和切换窗口，像xshell和CRT等工具对我来说就有些功能过冗余了。当然，像能和这两个工具结合在一起的文件传输插件还是挺不错的，不过也可以通过samba和IDE的remote devetop的sync功能将就将就，对于偶尔才会有代码以外的文件需要替换时，winscp足够使用了。

去年好像windows 10 开始官方提供OpenSSH支持，我立马也就升级了。不过很可惜在powershell和cmd环境中，像Ctrl+C等快捷键会直接被powershell给捕获，导致即便用这两个窗口ssh到了设备上，表现也并不是那么好。另外powershell和cmd的默认字体和高亮的美观度也不是很喜欢……

根据windows terminal的文档[^2]中所说的快捷键使用，基本上我用到的tmux功能都可以修改windows terminal的profile来模拟一致。且针对windows上需要临时复制单pane的内容时，这个相比putty能够不用单独全屏化窗口再复制，算是一个优势。

不过现在唯一的不足就在于，在这个版本暂时没有临时zoom全屏化单个pane的功能，根据[^1]中说的，`#996`暂时也没有纳入`1.x`的计划中，在`2.0`中才会发布。这就有点可惜了，只能再期待了。


## tmux

```
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

~/.tmux.conf
set -g mode-keys vi
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
​
set -g @continuum-save-interval '15'
set -g @continuum-restore 'on'
set -g @resurrect-capture-pane-contents 'on'
​
# Other config ...
​
run -b '~/.tmux/plugins/tpm/tpm'

```

## Reference
1. [Scenario: Add support for panes · Issue \#1000 · microsoft/terminal](https://github.com/microsoft/terminal/issues/1000)
2. [Windows Terminal Key Bindings \| Microsoft Docs](https://docs.microsoft.com/en-us/windows/terminal/customize-settings/key-bindings#tab-management-commands)
