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

## 清理
不能直接删除那个硬盘里的目录, 必须得`tmutils`来清理, 预计可能是增量备份. 我当时删的时候没意识到这点, 当时手动删的几个目录完全没被识别出已经被释放, 从而腾出空间.

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

## 长句 搜狗/百度输入法 输入长句时 有上限 尝试用原生

已给搜狗和百度输入法提供了反馈, 留了邮箱, 不知道有没有反馈了. 

* return可以在中文模式下 直接输入英文
* Caps才是切换中英文按钮

类似下面这俩微软输入法的问题一样.

[微软拼音长句输入字数限制问题](https://social.technet.microsoft.com/Forums/systemcenter/ro-RO/0653e7ed-2155-4842-baf3-66a64d88e25b/24494367192534038899382712147736755208372338325968384802104638?forum=2087)

[微软拼音长句输入字数限制问题](https://social.technet.microsoft.com/Forums/sqlserver/cs-CZ/0653e7ed-2155-4842-baf3-66a64d88e25b/24494367192534038899382712147736755208372338325968384802104638?forum=2087)



## 输入法 输入卡顿

[BigSur 自带中文输入法卡顿 \- V2EX](https://www.v2ex.com/t/731025)

跟这个表现很像. 卡顿的时候window server占用也高.

当前是Monterey 12.6版本.

看到这个愈发感觉还是搞个纯linux笔记本省事...

> 其原因似乎是 Google Chrome 浏览器自带的一个更新组件 Keystone 触发了 macOS 内部的某种 bug。有很多其他用户也都发现了这两者间的关联。触发这个问题并不要求 Chrome 正在运行，部分用户仅仅是安装 Chrome 就可轻易重现。

[降低 WindowServer 的 CPU 占用 \- My Nook](https://blog.mynook.info/post/macos-windowserver-calm-down/)


确实, 关了chrome, 立马表现就正常了...

开了chrome之后又复现了...

[1158402 \- Chrome/Keystone causing abnormally high WindowServer CPU usage when not running? \- chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=1158402#c18)

问题单仍旧在这里.

临时先切换成Edge了. 虽然vscode有我那个markless问题引起的卡顿, 但是应该没延迟到那个程度.




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

## 通过`Turbo Boost Switcher`暂时关闭睿频, 姑且在cpu满载100%的时候, 风扇不会那么容易转了

## 通过cpulimit对指定进程的cpu使用进行限制.

[opsengine/cpulimit: CPU usage limiter for Linux](https://github.com/opsengine/cpulimit)

[有没有办法限制某个程序进程的 CPU 占用率呢？ \- V2EX](https://v2ex.com/t/775774)

``` bash
for i in `ps -ef | grep ceph-osd | grep -v grep | awk '{print $2}' `; do nohup cpulimit -l 500 -p $i &>/dev/null & done 

for i in `ps -ef | grep ceph-osd | grep -v grep | awk '{print $2}' `; do kill -SIGCONT $i  & done  
```

> cpulimit works by continuously sending SIGSTOP and SIGCONT to the target process to limit it's cpu time. So the program in this case doesn't hang, it's doing it's job.

需要注意cpulimit进程退出时, 需要及时发送SIGCONT信号恢复

[linux \- cpulimit not working correctly \- Stack Overflow](https://stackoverflow.com/questions/21221343/cpulimit-not-working-correctly)

### 作废, cputhrottle的task_for_pid函数无法使用, appPolice无法显示出我想限制的指定进程
通过`appPolice`或者`cputhrottle`等限制指定进程的cpu
`cputhrottle`好像不能用, 源码编译后, `sudo`运行还是报这个

``` bash
cputhrottle libc++abi.dylib: terminating with uncaught exception of type Process::ManipulatorException: Error on task_for_pid
```

简单看[bash \- Iterate over pgrep results \- Stack Overflow](https://stackoverflow.com/questions/54242957/iterate-over-pgrep-results)
好像是`API`在高版本改了? 不能用这个了?


## app tamer for mac (cpu优化电池管理工具)

资源占用过高

)

# automator
## finder增加右键按钮

参考[在 Finder 的右键菜单中添加「Open in VSCode」 \| 始终](https://liam.page/2020/04/22/Open-in-VSCode-on-macOS/)

在`automator`中可以增加比如`open in vscode`的能力~

![](mac系统使用小记-md/mac系统使用小记-md_2021-09-16-23-33-29.png)

# android投屏

[\(1 封私信 / 20 条消息\) 如何将android手机屏幕投影至Mac？ \- 知乎](https://www.zhihu.com/question/38722634


## 基于DLNA的`Macast`
* 装到mac上之后, 可以直接投屏到mac上


## 万物互联 KDE connect (mac+android)

## QtScrapy 即Scrapy的图形化版本

[barry\-ran/QtScrcpy: Android real\-time display control software](https://github.com/barry-ran/QtScrcpy)
### 步骤
```
无线连接步骤（保证手机和电脑在同一个局域网）：
安卓手机端在开发者选项中打开usb调试
通过usb连接安卓手机到电脑
点击刷新设备，会看到有设备号更新出来
点击获取设备IP
点击启动adbd
无线连接
再次点击刷新设备，发现多出了一个IP地址开头的设备，选择这个设备
启动服务
备注：启动adbd以后不用再连着usb线了，以后连接断开都不再需要，除非安卓adbd停了需要重新启动
```


### 控制
MIUI注意除了USB调试还需要开启`USB调试(安全设置)`

# linux替代命令
## ldd == otool -L


# 网络

## 针对不同wifi, 使用不同dns

mac, 应该可以做到, 针对不同的wifi, 切换不同的dns服务器

> Mac下面的根据场景切换网络配置
> 我们只需要在连接一个新网络时，添加一个位置描述，然后跟之前一样设置各连接参数，然后应用。
> 
> 之后在场景发生变化后，如果想切换不同位置的网络配置时，在位置处选择你之前添加好的场景，然后应用就可以了


![](mac系统使用小记-md/mac系统使用小记-md_2022-02-17-16-00-17.png)

[macOS 自定义场景以快速切换不同的网络连接参数\_weixin\_33755847的博客\-CSDN博客](https://blog.csdn.net/weixin_33755847/article/details/91745270)

wifi profile switcher


[Mac OS 自动根据 WI\-FI 名字改变网络位置 \- Razeen\`s Blog](https://razeen.me/posts/auto-change-network-location-base-on-name-of-wifi/)



# 系统插件功能实现 hammerspoon

参照[hammerspoon扩展](hammerspoon神器扩展.md)方式配置.

# 硬件相关

## 触控板或者键盘操作一下, 屏幕下半个屏幕闪烁, 外接屏幕也不停刷新.

疑似某个软件引起的? 但是在家里又没有这个问题.

会不会是chrome开几百个窗口, 某个窗口引起的?

*重启设备后恢复*, 看起来还是跟软件有关.

## 左上侧的type-c口充不了电了




# 代理

## clash配置

配置proxy-groups设置`url-test`就自动选最快的了, 就不需要手动更改了. 不过注意需要把`直接连接`的这个代理从选择的这里去掉, 不然延时一般都是最低
``` yaml
proxy-groups:
  -
    name: 🔰国外流量
    type: url-test
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
    proxies:
      - '广东移动转台湾HiNet[M][倍率:1]'
      - '广东移动转新加坡Azure[倍率:0.9]'
      - '广东移动转日本NTT[倍率:0.8]'
```

## chrome代理问题

我看我的`switchOmega`已经停用很久了, 这次想考虑easyconnect和clash并存, 好像就要改浏览器的路由.

看到配置的地方了, 是在`Wifi`的高级选项里设置了的proxies, 这里增加了公司的url bypass之后能共存了.

# 录屏

## 终端录屏 asciinema


# 显示

## 分辨率 27寸 4K , 1920x1080感觉能显示的窗口太少

[Mac 4K 显示器 如何设置缩放？ \- V2EX](https://v2ex.com/t/759674)

按option显示成2304x1296 感觉字大小刚好, 又能显示多个窗口了.

# 邮箱

## qq邮箱 mail好像一改密码又登录不上了.

mac mail邮箱登录密码必须手动生成授权码

手动输入 且 敲回车

不能复制粘贴, 也不能直接点击验证/ok

[\(1 封私信 / 73 条消息\) mac邮件添加邮箱无法验证，求大神帮助？ \- 知乎](https://www.zhihu.com/question/42011333)

# 进程

## sysmond: activity monitor的监控进程.

# GNU tools

```
 brew install coreutils
 brew install findutils
 brew install gnu-sed
 brew install gnu-indent
 brew install gnu-tar
 brew install gnu-which
 brew install gnutls
 brew install grep
 brew install gzip
 brew install screen
 brew install watch
 brew install wdiff --with-gettext
 brew install wget
 brew install less
 brew install unzip


 export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/binutils/bin:$PATH"
export PATH="/usr/local/opt/ed/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/findutils/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/gnu-indent/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/gnu-tar/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/gnu-which/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/grep/libexec/gnubin:$PATH"
```

# 鼠标

## 滚动加速度 导致 鼠标滚动会跳来跳去

```

defaults write .GlobalPreferences com.apple.mouse.scaling 0
```

这条疑似有效?

[\(1 封私信 / 80 条消息\) mac os 苹果系统如何关闭鼠标滚轮加速？ \- 知乎](https://www.zhihu.com/question/68155111)


## parallel desktop 虚拟机内鼠标飘: 关闭对鼠标的游戏优化即可

# 任务栏

## 小图标过多 

对于不需要的, 直接`cmd`+拖拽下来就能删除该图标

比如搜狗输入法的图标就被我删了.

### Bartender bar

根据下面大佬安利的, 采用bartender 类似windows的小图标隐藏的效果一样, 可以折叠掉小图标. 

这里来看, 确实似乎不够人性化, 程序的应用菜单, 确实会占用不少屏幕, 这样任务栏对于需要大量插件的用户来说, 确实会产生冲突, 这种情况下应该提供一种折叠之类的方案来提供支持.

[macOS 顶上菜单栏空间不够，右侧小图标满了放不下了，一些图标直接显示不出来直接隐藏了，这种情况怎么解决呢？ \- V2EX](https://www.v2ex.com/t/521579)

[Mac 選單列空間不夠嗎？Bartender 讓你擁有第二選單，隱藏不需要的圖示 \- Rockyhsu](https://www.rockyhsu.com/bartender-4-review/)

# Command Line Tools
## python

``` bash

ll /usr/local/Cellar/python@3.*        
/usr/local/Cellar/python@3.10:
total 0
drwxr-xr-x 13 sean10 416 Oct  5 02:45 3.10.7

/usr/local/Cellar/python@3.9:
total 0
drwxr-xr-x 13 sean10 416 Sep 18  2021 3.9.7


➜  sean10.github.io git:(hexo) ✗ ll /usr/local/bin/python3*
lrwxr-xr-x 1 sean10 40 Oct  5 02:44 /usr/local/bin/python3 -> ../Cellar/python@3.10/3.10.7/bin/python3
lrwxr-xr-x 1 sean10 47 Oct  5 02:44 /usr/local/bin/python3-config -> ../Cellar/python@3.10/3.10.7/bin/python3-config
lrwxr-xr-x 1 sean10 43 Oct  5 02:44 /usr/local/bin/python3.10 -> ../Cellar/python@3.10/3.10.7/bin/python3.10
lrwxr-xr-x 1 sean10 50 Oct  5 02:44 /usr/local/bin/python3.10-config -> ../Cellar/python@3.10/3.10.7/bin/python3.10-config
lrwxr-xr-x 1 sean10 40 Sep 18  2021 /usr/local/bin/python3.9 -> ../Cellar/python@3.9/3.9.7/bin/python3.9
lrwxr-xr-x 1 sean10 47 Sep 18  2021 /usr/local/bin/python3.9-config -> ../Cellar/python@3.9/3.9.7/bin/python3.9-config


```

像这里看到的, 其实升级的时候, 旧版本也保留了.

但是发现`/usr/bin/python3`居然是个python3.8版本, 这里就需要确认下这个环境里之前是咋装进来的了.


``` bash
➜  sean10.github.io git:(hexo) ✗ pip3 --version            
pip 20.2.3 from /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.8/lib/python3.8/site-packages/pip (python 3.8)
➜  sean10.github.io git:(hexo) ✗ ll /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework                 
total 0
lrwxr-xr-x 1 root  24 Oct  5 02:40 Headers -> Versions/Current/Headers
lrwxr-xr-x 1 root  24 Oct  5 02:40 Python3 -> Versions/Current/Python3
lrwxr-xr-x 1 root  26 Oct  5 02:40 Resources -> Versions/Current/Resources
drwxr-xr-x 5 root 160 Oct  5 02:42 Versions
➜  sean10.github.io git:(hexo) ✗ ll /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions 
total 0
drwxr-xr-x 10 root 320 May  3  2020 3.7
drwxr-xr-x 10 root 320 Apr 30 13:40 3.8
lrwxr-xr-x  1 root   3 Oct  5 02:40 Current -> 3.8
➜  sean10.github.io git:(hexo) ✗ stat /usr/bin/python3
  File: /usr/bin/python3
  Size: 167120    	Blocks: 24         IO Block: 4096   regular file
Device: 1,9	Inode: 1152921500312781207  Links: 76
Access: (0755/-rwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/   wheel)
Access: 2022-08-24 16:59:39.000000000 +0800
Modify: 2022-08-24 16:59:39.000000000 +0800
Change: 2022-08-24 16:59:39.000000000 +0800
 Birth: 2022-08-24 16:59:39.000000000 +0800
```

根据这段判断, 比较像是CLT装进来的...但是时间戳和我今天装的好像对不上...怎么确认CLT不同版本里对应的python版本呢?


> From the Xcode 11 release notes...


> 
> ”In future versions of macOS, scripting language runtimes won’t be available by default, and may require you to install an additional package. If your software depends on scripting languages, it’s recommended that you bundle the runtime within the app.”
> 
> 
> 
> It might be available in the short term, but not the long term.



根据上面这句可以得知, CLT不再安装python3, 即可以通过brew或者pyenv管理了.

暂时先不管, 但是如果需要清理

* /usr/bin/python3
* /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions 


# OCR, 12版本支持了`实况文本`, 用mac原生软件预览图片时, 鼠标放过去就自动OCR了.

### 是否可以直接弹出预览的窗口, 而不要二次手动打开?

#### 通过hammerspoon, 绑定一个快捷键, 截图后自动调用preview去打开预览的图片?



# 显示器

### 12.6 升级大概2周后, 外接显示器忽然没信号了.

睡眠后, 尝试激活, 发现无信号了.

#### 解决方式: 拔掉显示器电源几十秒后, 恢复了.


#### 已尝试
1. type-c转dp线, 插其他人mac和显示器上通过
2. mac接其他人的拓展坞, 走htmi通过
3. 其他mac接这个type-c转dp和显示器你 ,依旧无信号.

#### 怀疑点
1. 这台电脑的type-c转dp协议出问题
2. 显示器整体出问题
3. 显示器的dp口出问题

下一步尝试type-c转hdmi线, 公司有个, 主要需要出去拿.


# airpods 已修复 支持记忆非ios设备或者ios设备上上一次连接设备

[AirPods Pro won't autoconnect to Anroid Phone : airpods](https://www.reddit.com/r/airpods/comments/qcm4sg/comment/i8y12i9/)

[AirPods Pro won't autoconnect to Anroid Phone : airpods](https://www.reddit.com/r/airpods/comments/qcm4sg/comment/i8y12i9/)


# 投屏 
## ios设备, 升级即可mirror

## android 设备  QtScrcpy即可

## 显示同步(没有难度, 随便点一键连接也可以)

wifi连接的延时稍高一点, 还是usb连接比较稳定.

### 音频同步
点击`安装sndcpy`, 会报`AudioOutput::" "/bin/bash: sndcpy.sh: No such file or directory\n"`

["AudioOutput::" "/bin/bash: sndcpy\.sh: No such file or directory\\n" · Issue \#643 · barry\-ran/QtScrcpy](https://github.com/barry-ran/QtScrcpy/issues/643)

参考这个["AudioOutput::" "/bin/bash: sndcpy\.sh: No such file or directory\\n" · Issue \#643 · barry\-ran/QtScrcpy](https://github.com/barry-ran/QtScrcpy/issues/643), 接着usb线的情况下, `cd /Applications/QtScrcpy.app/Contents/MacOS && /bin/bash sndcpy.sh`即可成功安装. 然后就点击开始音频即可.



## 电池

![[Pasted image 20230218225149.png]]

根据这个来看, design好像是5000, 差的有点多.


## dropbox 同步问题

自从升级了Dropbox到了CloudStorage之后, 目录迁移到`/Users/sean10/Library/CloudStorage/Dropbox`之后, 出现了老是在下载云端文件的情况.

我期望的使用方式是跟之前版本一样, 没有变化时就是操作本地文件, 比如我外部打开整个目录, 直接触发全目录搜索就可以了, 不需要做什么多余的事情.

但是目前的体验就是会出现一直在转圈, 等待从云端同步.  如果真的要做这种逻辑, 那这种在云端的盘还不如hdd呢...

[No longer have offline access automatically? \- Page 5 \- Dropbox Community](https://www.dropboxforum.com/t5/View-download-and-export/Request-All-files-available-offline-by-default/m-p/644830)

这里看到的讨论是都遇到了相同的问题.




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