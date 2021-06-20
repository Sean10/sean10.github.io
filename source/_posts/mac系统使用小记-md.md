---
title: mac系统使用小记.md
subtitle: mac
date: 2021-01-16 19:36:07
updated:
tags: [mac]
categories: [专业]
---


# Time Machine

## mac time machine限速

> 苹果官方直接就有给出解决方案，关闭限速即可
``` bash
sudo sysctl debug.lowpri_throttle_enabled=0
```

> 如果想要再开启，输入以下命令即可
``` bash
sudo sysctl debug.lowpri_throttle_enabled=1
```

## 查看time machine日志
``` bash
sudo less +F "/Volumes/MacBackup/Backups.backupdb/MacBook Pro/2020-08-05-163227.inProgress/.Backup.618330747.626060.log"
``` bash

[macbook pro \- Time Machine in the "Cleaning Up\.\.\." state forever \- Ask Different](https://apple.stackexchange.com/questions/382772/time-machine-in-the-cleaning-up-state-forever)


## time machine目录terminal无权限

在Security * Privacy的Privacy中放开对Full Disk Access的Terminal权限.

# 文件系统

## APFS

> 稀疏文件、改进的 TRIM 操作，内建对扩展属性的支持
> 空间共享
> 数据加密
> 大小写敏感

### Volume
跟`lvm`那些的逻辑卷是不是差不多呢. 其中最大的亮点功能因为是不同卷组之间共享总体空间的功能了, 看上去应该是依托`COW`实现的. 但是具体机制不知道有没有哪篇文章提到.

不知道`Container`和`Volume`两层分别是起什么样的作用呢?

### firmlink
跟
``` bash
 ✗ cat /usr/share/firmlinks                  
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
/System/Library/Caches	System/Library/Caches
/System/Library/Assets	System/Library/Assets
/System/Library/PreinstalledAssets	System/Library/PreinstalledAssets
/System/Library/AssetsV2	System/Library/AssetsV2
/System/Library/PreinstalledAssetsV2	System/Library/PreinstalledAssetsV2
/System/Library/CoreServices/CoreTypes.bundle/Contents/Library	System/Library/CoreServices/CoreTypes.bundle/Contents/Library
/System/Library/Speech	System/Library/Speech
/Users	Users
/Volumes	Volumes
/cores	cores
/opt	opt
/private	private
/usr/local	usr/local
/usr/libexec/cups	usr/libexec/cups
/usr/share/snmp	usr/share/snmp
```


## Mac journal extended

# 目录结构[^4]
> Mac 根目录下有以下几个文件夹：
> /System 文件夹，系统文件夹。与Windows 之中的 C:\windows32 等文件夹类似。
> Library 系统资料库，其中的 Caches 可以删除。
> iOSSupport 提供了系统的 iOS 支持。
> /Applications GUI软件文件夹，共享的所有软件包都存放在此。
> /Library 应用资料库，包括了大部分非核心的系统组件。Caches 可删除。
> /Users 文件夹，与 Linux 之中的 /home 文件夹功能类似。而mac 之中的 /home 只是为了与 Linux 兼容，一般不放任何东西。
> /Network 和 /net 网络相关，空的。
> /Volumes 与 /mnt 类似，其中挂载了全部硬盘、网络硬盘等。
> /sbin，/bin，/usr /dev文件夹，与 Linux 基本一致。与 Linux 兼容。
> /etc, /var /tmp 文件夹，是位于 /private 之中对应文件夹的软连接。存放系统配置、数据库、缓存等。用于与 Linux 文件结构兼容。
> 注意，/root, /procfs, /boot, /sysfs 等非必须文件夹均不存在。

遵照`freeBSD`的`/bin,/etc,/lib`目录都是不建议修改的, 所以所有程序都是装到`/usr/local`目录下

# 开发调试 clang相关

## lldb
`lldb -c /cores/core.99415` 这样就可以调试了,不需要指定可执行文件看起来

# brew 包管理

## brew使用



``` bash
#更新brew到最新版本
brew update-reset

# 显示这个包内安装的文件的路径。
brew list redis

brew cleanup
# 查包的依赖包
brew deps --installed --tree
# 卸载包及其依赖
brew rmtree graphviz
```
### brew 运行详情


[Unsupported special dependency :maximum\_macos · Issue \#38604 · Homebrew/homebrew\-core](https://github.com/Homebrew/homebrew-core/issues/38604)

[macOS 使用 Homebrew 的经验分享 \| HelloDog](https://wsgzao.github.io/post/homebrew/)

没在文档里找到如何下显示brew update的进度，不然老是以为阻塞了，还好我想起来一般开源软件都用verbose，就试了一下，看了brew的写法也是相当不错的。

不过就是也只能显示到fetching的进度了，fetching快不快应该只能靠监控流量了。

## 字体[^5]
* fair code
    * `brew tap homebrew/cask-fonts`
    * `brew cask install font-fira-code`


## brew源配置

mkdir -p $(brew --repo homebrew/core)

git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
git -C "$(brew --repo homebrew/cask-fonts)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-fonts.git
git -C "$(brew --repo homebrew/cask-drivers)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-drivers.git


brew底层用的还是Git,git现在通过改host还是不怎么能提速，虽然现在用不了proxychains4，但是似乎可以直接用`git config --global http.https://github.com.proxy socks5://127.0.0.1:1086`直接用代理，唉，早知道这个就没那么多事情了。

奇怪好像找到我的cask和homebrew不正常的原因了, 不知道为什么, 我执行上面的这堆命令的结果, git和目录的对应都是错的. cask的目录里的git却是什么Font的.

/usr/local/Homebrew/Library/Taps/homebrew

奇怪, 怎么好像这个目录下属的子目录里, 还是在brew那个仓库里呢... 是因为装的太早, 不是通过tap安装的? 

看起来的确是这个问题, 新装的这个version目录没问题

brew tap homebrew/core
brew tap homebrew/cask
brew tap homebrew/cask-fonts
brew tap homebrew/cask-drivers
brew tap homebrew/cask-versions

暂时不配置源了, 用代理下载也挺快.
[安装HomeBrew 失败的解决方案\(Error: Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew\-core failed\!\) \- 子钦加油 \- 博客园](https://www.cnblogs.com/zmdComeOn/p/11990079.html)

## 老是报caskroom/homebrew-cask卡住

实际上不存在这个了, 已经改名成homebrew/homebrew-cask了.

[macos \- Error: caskroom/cask was moved\. Tap homebrew/cask\-cask instead \- Stack Overflow](https://stackoverflow.com/questions/58335410/error-caskroom-cask-was-moved-tap-homebrew-cask-cask-instead)
# 降频

> 机时以 sudo 运行 Turbo Boost Switcher，就不用来回输密码切换了。
> 
> 如命令行输入：sudo /Applications/Turbo\ Boost\ Switcher.app/Contents/MacOS/Turbo\ Boost\ Switcher

[Turbo Boost Switcher 真香，试用了一天立刻买 Pro 了 \- V2EX](https://fast.v2ex.com/t/545553)

# Spotlight
## 快捷键

Spotlight打开之后，输入单词，按command+b可以直接用浏览器搜索，用command+L可以直接跳到字典项进行查询。

# 购买+apple care+
从官网买的时候, 听说官网不像渠道那边, 以开机联网开始激活, 而是以发货时间开始,如果发货到你收货就过了3天, 那就相当于你的保修期已经过了3天, 对于我想从淘宝加3天内的`apple care+`的需求来说, 这就导致我需要提前查询到序列号.

还好, 虽然大部分地方没提到,但是实际上只要进入发货阶段, 当你邮箱里收到发货信息之后, 序列号已经有了, 可以直接联系客服, 通过一些信息直接询问序列号. 根据客服当时说, 根据正常流程, 发票实际上会自动发出, 只是可能相比你直接去问要晚一点发送. 我问到序列号, 办完`apple care+`后的一天收到了发票.

# launchctl

如何通过launchctl控制进程呢?比如App

其实好像也只有pkill一条路.

# touchbar
一开始我以为必须按照他的提示来使用以前的那些功能按钮.

偶然按了fn的情况下去按touch bar上显示的f12, 发现也能成功调整音量. 所以如果知道原来这个按钮上对应的功能键, 其实还是可以用的. 

# 关闭系统更新提示[^6]
> 3) Paste this command in the Terminal window, then press Enter to execute it:
> 
> sudo softwareupdate --ignore "macOS Catalina"
> 
> 4) Next, paste this in Terminal and press the Enter key to run the command:
> 
> defaults delete com.apple.preferences.softwareupdate
> 
> 5) Lastly, execute the following command in Terminal:
> 
> softwareupdate --list


# 休眠时, 你的前台程序会收到的信息


## 系统休眠过程中主要是IO超时中断,这部分做过滤处理就好了

CPU会被暂停, 所以可能一些程序如果没做IO超时处理,就会直接触发异常中断了.

的确很对, 像是一些视频网站打开后休眠再次打开, 缓存的连接都失效了, 一般需要刷新再操作了.

# 输入法(中英文混合输入)
我习惯用搜狗输入法了, `高级->动态组词`, 勾选上之后, 单纯输英文时, 一般都不会显示出中文的候选词了, 直接空格就能够输入了.

## 在`Ctrl + s`弹出的`save`窗口里, shift切换中英文失效[^7]

据说是微信的客户端引起, 重启设备似乎就好了.

我的确打开过微信, 然后没重启过.

试试.

# mac不自动连接热点

`10.15`版本, 我点开wifi管理, 发现居然直接就有一个`Automatically join this network`的选项,而我没有勾选这个.

之前我用`10.12`版本的时候, 我记得网络设置那里并没有这样这个选项, 导致每次都需要我手动点击. 现在居然完美解决了这个问题...

# vmware虚拟机不自动休眠[^8]未成功

目前遇到的主要问题是, 我开的linux虚拟机, 用来起`Docker`的虚拟机, 老是在我电脑合盖之后, 就自动休眠了.

找了下, 搜关键词`stop vmware from hibernating linux`搜到了.

在虚拟机的`vmx`文件中添加`suspend.disabled = "TRUE"`

emm. 似乎无效.

# 合盖掉电问题, 基本一个晚上掉50%
```
pmset -g log | grep Wake | less
并没有看到有在睡觉期间被激活的日志...

pmset -g custom

主要设置了这个
sudo pmset -b tcpkeepalive 0

因为我试了下, 昨天关Wifi之后耗电就少了很多. 所以试试这个. 本身我合盖以后就对他的自动下载能力不抱期望.
```

# BT transmission的tracker添加
主要用的这个`[GitHub \- blind\-oracle/transmission\-trackers: Script to automatically add trackers from a list to all torrents in Transmission](https://github.com/blind-oracle/transmission-trackers)`


# CPU/GPU
## windowsserver的CPU占用高
这个是mac的图形界面展示的进程, 因为是集成显卡, 所以集显能力不足的时候, cpu占用会偏高.

有人说是降低透明度就可以不卡顿, 不知道是否有效.

> 在System Preferences > Keyboard中， 将Key Repeat跟Delay Until Repeat往左边设置：

有人说这样设计也能好转. 姑且看看吧.[^11]


# Reference
1. [当 Mac 升级到 Catalina 时，苹果在硬盘里施了点魔法 \- 少数派](https://sspai.com/post/57052)
2. [闲聊ReFS与APFS \- 知乎](https://zhuanlan.zhihu.com/p/30721313)
3. [Apple新发布的APFS文件系统对用户意味着什么\-InfoQ](https://www.infoq.cn/article/2016/07/apple-apfs)
4. [Layton's Blog \- 技术摘要\| Mac OS 与 Linux 的目录结构比较](https://www.laytonchen.com/post/tech/macandlinuxdict/)
5. [Programming Fonts \- Test Drive](https://www.programmingfonts.org/)
6. [Apple's has brought back the nagging — you can no longer ignore major macOS updates](https://www.idownloadblog.com/2020/05/28/apple-removes-ignoring-macos-updates/)
7. [\(24 条消息\) Macbook中英切换键失效，怎么办？ \- 知乎](https://www.zhihu.com/question/338779279)
8. [Disabling the suspend feature for a virtual machine in VMware Fusion and VMware Workstation \(2056501\)](https://kb.vmware.com/s/article/2056501)
9. [macOS 电源管理修复 MacBook 休眠耗电大问题 \- Marco Nie](https://blog.niekun.net/archives/1622.html)
10. [BT种子获取更多连接的方案（增加trackerslist） \| Boris的备份库房](https://boriskp.github.io/trackerslist/)
11. [Mac系统WindowServer进程占用CPU资源问题 \| Hanjie's Blog](http://www.luohanjie.com/2017-01-16/mac-windowserver-process-cpu-resources.html)