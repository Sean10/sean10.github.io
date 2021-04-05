---
title: arch体验
subtitle: arch
date: 2021-01-02 23:03:56
updated:
tags: [arch, laptop]
categories: [专业]
---

## 背景

有点用腻了mbp的mac os系统的感觉,想换arch了.

因为总觉得mac系统里好乱, 好多感知不到的空间管理. 而且终究只是unix, 并不是linux. 

但是嘛, 都用了一段时间之后回来想想, 最近我比较倾向于`arch`的原因主要是最近工作所需一直在用`linux`规范下的`systemd`控制下的`CentOS`, 用的比较熟悉了, 对目录结构比较清楚了, 而对于自己非常不熟悉的`freeBSD`基础的mac, 有种不在自己控制内的感觉...

现在想了想, 这其实不就是舒适区的问题吗? 学习一下mac的目录结构和日志排查等, 其实也并不会有多费事, 也并不会增加我记忆`unix`生态的混淆程度.

哎,但是看笔记本的品控上来说,可能还是mac os让人比较省心了. 至少售后给的新笔记本不会像xps那样经常出问题...

## 硬件

### XPS(arch推荐比较多的是这个,但是因为品控问题不推荐的也是这个)
什么电流音之类的,决定还是不选这个了.

虽然好像XPS 7590 15寸或者13寸的直接支持hdmi 2.0接口,不过品控还是太让人担心了.

### thinkpad x1c
emm, 似乎还行,但是触控板的体验还是比不上mac的应该

### matebook 
好像散热不行, 键盘部分会烫?

听说bios不太行?



# 安装踩坑
## manjaro踩坑
### `bootloader`一直没能正常使用
参照[^3]这个带图片的安装过程终于安装成了.
发现我直接选择第三种`systemd-boot`的引导,最后是看不到进入系统的东西的,可能兼容性上有问题? 最后我额外安装了`Refind`这套引导,终于进入桌面了.

## arch安装
### todo 
大小写老师被自动切换了

似乎是vmware 15.5这个版本的缺陷

> Temporary solution for Ubuntu guest which worked for me was just disabling the Caps Lock key all together with this

> setxkbmap -option caps:none
setxkbmap -option caps:none # (disable the caps lock key)

xdotool key Caps_Lock # (toggle caps lock)
[Caps Lock Issues With Upgrade \- VMware Technology Network VMTN](https://communities.vmware.com/t5/VMware-Workstation-Pro/Caps-Lock-Issues-With-Upgrade/td-p/2285920)

禁用这个按钮倒也是个方案, 反正我基本也不用.

不过就怕他也影响宿主机, 那就很讨厌了.

### 为啥我安装的虚拟机鼠标放在Linux界面再挪出去，大小写键盘总是自动切换呢？用的vmware ubuntu.如图
[为啥我安装的虚拟机鼠标放在Linux界面再挪出去，大小写键盘总\_虚拟机吧\_百度贴吧](https://tieba.baidu.com/p/4211342546)
[VMware虚拟机中大小写不停切换的问题\_helen2977的博客\-CSDN博客](https://blog.csdn.net/helen2977/article/details/82143556/)
好像无解

### vmware-tools安装运行

我之前应该是直接`pacman -Syy open-vmware-tools`

现在发现`vmware-tools`这个服务没能启动.

好像搜到说是

[\[SOLVED\] Automating installation of vmware\-tools](https://www.linuxquestions.org/questions/linux-general-1/automating-installation-of-vmware-tools-4175446206/)

实际上我装的是`open-vm-tools`
### open-vm-tools
`vmtoolsd`这个服务似乎可以工作

[VMware中的Manjaro启用复制粘贴\_darkula的博客\-CSDN博客](https://blog.csdn.net/darkula/article/details/107073762)

> Installation
> Install open-vm-tools. If the legacy vmhgfs shared folder module is desired, the open-vm-tools-dkmsAUR package must be installed (the new vmhgfs-fuse driver is included in open-vm-tools). Start and/or enable vmtoolsd.service and vmware-vmblock-fuse.service.

> Try to install gtkmm3 manually if it does not work properly. To enable copy and paste between host and guest gtkmm3 is required.

[VMware/Install Arch Linux as a guest \- ArchWiki](https://wiki.archlinux.org/index.php/VMware/Install_Arch_Linux_as_a_guest#Installation)

不能复制粘贴的时候, 不知道为什么,执行下这个`vmware-user`就能用了.

[Arch Linux / Manjaro 配置 VMware copy/paste](https://zzz.buzz/zh/2018/05/08/vmware-copy-paste-for-arch-linux-manjaro/)

vim内没开全局剪切板, 无法向外复制. cat处理吧.

### 重点

``` bash
# 这里fdisk分区时, 及时将类型修改为EFI类型
# 否则会出现错误: cannot find a GRUB drive for 

mount /dev/mapper/VG_0-root /mnt
mkdir /mnt/boot
mount /dev/sdb1 /mnt/boot
swapon /dev/mapper/VG_0-swap

timedatectl set-ntp true
vim /​etc/​pacman.d/​mirrorlist
pacman -Syy
pacstrap /mnt base base-devel linux linux-firmware man-db man-pages iwd lvm2 dhcpcd vim systemd


genfstab -U /mnt >> /mnt/etc/fstab


 arch-chroot /mnt

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

/etc/locale.conf
locale-gen

# 修改
/etc/mkinitcpio.conf
HOOKS=(base systemd ... block sd-lvm2 filesystems)
#Initramfs
mkinitcpio -P


# 安装grub
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
# 注意要挂载/boot/efi，见上面挂载boot分区那步
grub-mkconfig -o /boot/grub/grub.cfg

这次
useradd -m -G wheel cc
passwd cc
nano /etc/sudoers
在 root ALL=(ALL) ALL 下面添加
用户名 ALL=(ALL) ALL
为你刚才创建的用户 添加sudo权限

pacman -S xorg-server
pacman -S i3-wm

# useradd的用户无法启动时, 安装一下这个, 创建一下自己这个用户的配置文件就好了.
pacman -S xorg-xinit

复制 /etc/X11/xinit/xinitrc 到～/.xinitrc。注释掉文件后面的最后的以下几行。

twm &
xclock -geometry 50x50-1+1 &
xterm -geometry 80x50+494+51 &
xterm -geometry 80x20+494-0 &
exec xterm -geometry 80x66+0+0 -name login

然后添加i3启动命令

exec i3

sudo pacman -S i3
sudo pacman -S wqy-microhei adobe-source-code-pro-fonts



# i3 Error: Status_command not found (exit 127) : linuxmint
[\(1\) i3 Error: Status\_command not found \(exit 127\) : linuxmint](https://www.reddit.com/r/linuxmint/comments/3f9k9s/i3_error_status_command_not_found_exit_127/)
pacman -S i3status

#HIDPI
[HiDPI \(简体中文\) \- ArchWiki](https://wiki.archlinux.org/index.php/HiDPI_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E9%9D%9E%E6%95%B4%E6%95%B0%E5%80%8D%E7%BC%A9%E6%94%BE%E4%B8%8B%E7%9A%84Bug)

Simplified appliance containg one one-liner

eval $(cvt 2220 1250  60 |sed 's/Modeline/xrandr --newmode /g'|sed -n '1!p')

as a proper result resolution screen size aspect ratio might be afterwards reevaluated/adjusted, therefore find out the created resolution by xrand command - appended in the end of output,

1) assign the resolution to a specific display -

xrandr --addmode VGA-1 "2224x1250_60.00"

2) output the desired resolution on the display

xrandr --output VGA-1 --mode "2224x1250_60.00"

To list all installed shells, run:

$ chsh -l
And to set one as default for your user do:

$ chsh -s full-path-to-shell

if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
  exec startx
fi

yay-git.git

makepkg -si

# 配置dhcpcd自动

systemctl enable dhcpcd

退出startx方式
win+shift+e退出桌面, 这个方式好像会卡住, 输入不来哦东西.



```

按照这个终于过了.
[Arch Linux \(UEFI with GPT\) 安装 \| 沈煜的博客](https://shenyu.me/2020/04/11/arch-uefi-install.html)


[ArchLinux图形界面安装与美化：i3\+polybar\_盐焗咸鱼的博客\-CSDN博客\_archlinux安装i3桌面](https://blog.csdn.net/qq_33215865/article/details/90288997)

## 休眠


### systemd-swap

> Can we use this to enable hibernation?
> A: Nope as hibernation wants a persistent fs blocks and wants access to swap data directly from disk, this will not work on: swapfs

emm, 不能用这个来做休眠用的swap.

### 手动创建
``` bash
fallocate -l 16G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

filefrag -v /swapfile
GRUB_CMDLINE_LINUX_DEFAULT="quiet resume=/dev/sda1 resume_offset=313344"
grub-mkconfig -o /boot/grub/grub.cfg
/etc/mkinitcpio.conf
mkinitcpio -P
```
[Arch Linux 使用 Swap File 进行休眠 \- xzOS](https://xzos.net/arch-linux-hibernation-into-swap-file/)
## 安装vmware

[VMware \(简体中文\)/Installing Arch as a guest \(简体中文\) \- ArchWiki](https://wiki.archlinux.org/index.php/VMware_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)/Installing_Arch_as_a_guest_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

### systemd-boot和 grub

[systemd\-boot和EFISTUB \- 知乎](https://zhuanlan.zhihu.com/p/20623053)
### boot manager可以看到有我新建的grub, 但是EFI似乎没法直接从硬盘启动?

### UEFI启动
/etc/locale.gen
### UEFI和 grub什么关系?


#### 似乎我的/boot分区划分出问题了, efi无法找到s
并不是EFI分区, 导致bootctl无法识别了.

bootctl status可以帮助判定

[\[SOLVED\] EFI partition not detected / Installation / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=229994)

grub-install --target=i386-pc /dev/sda
用legacy就尅引导了

用uefi的反而麻烦, 必须分区时gpt符合efi的

### 分区lvm
[Installation guide \- ArchWiki](https://wiki.archlinux.org/index.php/Installation_guide)

#### lvm不能直接被引导?

#### boot分区独立分区的意义?
创建出来之后, 从哪里复制内容过来, 好像这个文档里都没有提啊s

[Partitioning \(简体中文\) \- ArchWiki](https://wiki.archlinux.org/index.php/Partitioning_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%A0%B9%E5%88%86%E5%8C%BA)

/boot分区里的内容在哪里呢?

使用过grub-config来往里面生成内容?

mkinitcpio -P 似乎也能生成新的initramfs

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub  
grub-mkconfig -o /boot/grub/grub.cfg



[将已经存在的archlinux迁移到lvm \- 知乎](https://zhuanlan.zhihu.com/p/130445778)
#### initramfs选择?




[LVM \(简体中文\) \- ArchWiki](https://wiki.archlinux.org/index.php/LVM_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
[archlinux安裝手记（Win10\+Arch、GPT\+UEFI、lvm） \- 停止使用的账户 \- 博客园](https://www.cnblogs.com/unkownarea/p/6472461.html)

# 使用
## 源
``` bash
sudo pacman-mirrors -c China
pacman -Syy
pacman -Syu
好像说是命令是pacman -Syyu
pacman -Qi linux
pacman -Qs 在已安装中查询
pacman -Ss 在仓库中查询
pacman -R 卸载
pacman -Ru 循环卸载系统里不依赖的东西 

```

### pacman 升级python, 似乎没删掉旧内容?
* todo
  *  从`3.8`升级到`3.9`, 看了下`/usr/bin`下的确只有新的`python3.9`了, 但是`/usr/lib64/`下还存在`3.8`的目录, 是不是之前pip装的东西没有被自动卸载或者移植?

这是一个疑问句
> 不要用 pip 管理 /usr 下的包，它会和 pacman 打架。要么建 venv 在里边用 pip，要么不用 pip 只用 pacman。

根据这个[pacman 安装的python包无法识别 / 应用程序与桌面环境 / Arch Linux 中文论坛](https://bbs.archlinuxcn.org/viewtopic.php?id=10283), 是建议通过`venv`来管理python依赖的.


### 更新python之后, i3-sensible-terminal无法工作了

从`TTY2`执行了下`i3-sensible-terminal`, 结果发现是`psutils`这个库没了, 应该就是我刚才升级引起的了.

怎么让pacman来安装包呢?
``` bash
pacman -Syy python-pip
pacman -Syy python-six
```
装完`pip`之后报没有`six`, 安装了下就能使用了.

执行`terminator`, 报`You need to install gobject, gtk, pango to run Terminator`

最后, 我也没找到怎么处理的办法, 就把python3.8里的库复制到了python3.9的里...

### timeout

[\[Solved\] Connection time\-out on installation \(curl / iPv6 problem\) / Installation / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=174752)
捣鼓了半天, 最后是因为我在Arch里默认开了代理, 代理好像有问题了.

## 内核异常
### 卡在loading initial ramdisk
> There was a kernel update, some element of your initramfs wasn't successfully built and copied to your bootloader. Generally the output of mkinitcpio shows errors, chroot in and make it again.
执行
``` bash
mkinitcpio
# 就会报出initramfs的错误.
```
[\(1\) Stuck on Loading initial ramdisk : archlinux](https://www.reddit.com/r/archlinux/comments/58an6s/stuck_on_loading_initial_ramdisk/)

#### 打开调试日志

``` bash
vim /boot/grub/grub.cfg
# 去除quiet和设置log_level=7,rd.log=all
# 奇怪,怎么没生效呢?
```

通过增加`set debug=all`和`set ignore_log_level=all`倒是生效了, 看到了一些代码的执行记录. 但是倒是没看到Error信息

[General troubleshooting \- ArchWiki](https://wiki.archlinux.org/index.php/General_troubleshooting#Boot_problems)

> 为其他不是当前正在运行的内核创建镜像，添加内核版本到命令行， 可以在/usr/lib/modules目录查看支持的内核版本.
``` bash
 # mkinitcpio -p linux
 就会以当前安装的linux版本对/boot目录下的initramfs-linux.img进行再次生成了.
```
捣鼓了2天, 换了个稳定版的内核就正常开机了.
#### 进入reach target basic system之前的shell
在`linux16`那行追加`rd.break=pre-trigger`

在`linux16`这行追加`rd.debug`可以打出`initrd`中的`dracut`脚本的内容


## 周期内核升级
``` bash
pacman -Sy linux-lts
# 下面这个似乎只生成当前运行的内核的initramfs, 如果要生成指定, 需要指定preset
 mkinitcpio -P
# 指定内核版本生成?


 grub-mkconfig -o /boot/grub/grub.cfg


```

## terminal
### i3-sensible-terminal

* Mod+shift+q 退出程序

#### 快捷键 shortcuts
不知道为什么`alt+f`和`alt+b`没办法快速在单词之间跳

这个快捷键是`emulator`的还是`shell`的呢?

bash默认是启动了`emacs`模式的.

哦, 好吧, 是`Super`key覆盖掉了`alt+f`这个操作, 所以把`Super`key换成command也就可以了.

[\(1\) Can I completely swap the Alt and Super keys? : i3wm](https://www.reddit.com/r/i3wm/comments/bakkpg/can_i_completely_swap_the_alt_and_super_keys/)

`set $mod Mod4`

[Archlabs\-i3wm/config at master · alexandrebobkov/Archlabs\-i3wm](https://github.com/alexandrebobkov/Archlabs-i3wm/blob/master/config)

主要是我之前生成的配置文件里居然都是直接`Mod1 + Enter`之类的, 而不是`$mod + Enter`, 导致我直接`set $mod`没有效果.

## 网络
systemd-networkd
禁用ipv6
net.ipv6.conf.all.disable_ipv6 = 1

## 输入法

## kde最稳妥

可以和i3结合?

bspwm
## 文件系统只读问题
一般都是fstab被改坏了之类的.


## 桌面:i3wm
毫无疑问,我主要感兴趣的点就是能够尽量完全使用键盘来控制的方式.

$mod + Enter	启动虚拟终端
$mod + A	焦点转义到父窗口上


### i3lock
歧义性好严重...没有看到输入密码的框, 原来是因为默认就处在接收密码的状态下, `verifing`和`wrong`的状态就是密码验证的结果...
### 登录桌面[deprecated]
最后因为`vmware`里似乎`X11`的直接配置没能配置生效, 导致sddm始终分辨率都是`800x600`, 我最后还是选择使用`.xinitrc`来调用`xrandr`修改分辨率

```
pacman -S sddm
systemctl enable sddm #


vim /etc/sddm.conf.d/hidpi.conf
# 虽然我通过`pacman -Ql sddm`查到我装的位置是在`/usr/lib/sddm`里
[Wayland]
EnableHiDPI=true

[X11]
EnableHiDPI=true
```
### urxvt
rxvt-unicode
不知道这个好不好用

[\(1\) Which terminal simulator do you use with i3, and why? : i3wm](https://www.reddit.com/r/i3wm/comments/4f55fr/which_terminal_simulator_do_you_use_with_i3_and/)

urxvt 要支持hidpi, 还需要一些特殊配置. 先这样用吧. 已经挺好了.

#### 默认的复制粘贴

`Ctrl+alt+c/v`

#### unable to load base fontset
因为`HiDPI`, 所以增加了`pixelsize`的设置, 结果无法启动了.

看了下, 似乎是缺字体?




### 图片和壁纸 feh

feh

feh --bg-scale new.jpg

[feh \(简体中文\) \- ArchWiki](https://wiki.archlinux.org/index.php/Feh_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

### HiDPI[^7]
看起来多屏高分屏支持有点费事, 暂时还是不捣鼓了.

> xrandr --output eDP-1 --auto --output DP-1 --auto --scale 2x2

这个的scale 和那个~/.Xresources的Xft.dpi=192区别?

是不是xrandr修改了dpi之后, 修改指定程序的字体大小就行了?

`xrandr --dpi`这个好像是`xft.dpi`的更新的方案

这个设置了以后, 标题栏的确hidpi了, 但是应用程序并没有, 不管是Terminal还是chrome.

### .Xresources[^8]
Xft.dpi:   141
xrdb ~/.Xresources
xrdb -merge ~/.Xresources
xrdb -query -all

### 最后成功版本[^9][^10]
最后其实还是按照wifi操作成功的.

我在 .xinitrc中添加xrandr --output Virtual-1 --mode 2560x1600
在.Xresources中添加
```
Xft.dpi:  256
export GDK_SCALE=2
export GDK_DPI_SCALE=0.5
export QT_AUTO_SCREEN_SCALE_FACTOR=1
```
这次重启之后终于终端字体大小什么的都对了.

## 远程控制: x11远程和本地渲染不同?
我看到的窗口和颜色不太一样啊,vnc

# manjaro
先用i3试试.

# i3wm 使用
[^4]可以看. 不配置`de`还挺舒服的.


## 程序启动器
### dmenu
和`dmenu-manjaro`冲突

# Wayland仍未成熟

目前来看,虽然Wayland还是在逐渐开发, 不过Xorg的成熟度还是远比其成熟, Wayland存在的问题稍多.

## Sway
等价于`i3wm`的管理器,不过这个没能做到`i3polybar`的程度, 而且也始终是存在部分问题的. 所以如果要用arch, 还是继续`Xorg`下吧.
## 开关机
`shutdown -h now`

# 驱动
[【openSUSE】软件源和软件搜索 \-\-\- 看了之后 受益匪浅 \- 孙愚 \- 博客园](https://www.cnblogs.com/51linux/archive/2011/09/07/2169961.html)

# Reference
1. [Installation guide \- ArchWiki](https://wiki.archlinux.org/index.php/installation_guide)
2. [i3 \(简体中文\) \- ArchWiki](https://wiki.archlinux.org/index.php/I3_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
3. [Manjaro\-architect 安装指南\_兴趣斗士的博客\-CSDN博客\_manjaro architect](https://blog.csdn.net/BjarneCpp/article/details/95861978)
4. [i3: i3 User’s Guide](https://i3wm.org/docs/userguide.html#_default_keybindings)
5. [xps13\(9370\) Linux之路 · Kevin的笔记](https://jacksonpuppy.com/2019-07-21/xps13-linux.html)
6. [screen lock / verifying : i3wm](https://www.reddit.com/r/i3wm/comments/630r1r/screen_lock_verifying/)
7. [Linux 下， hipdi 高分屏外接显示器显示怎么整啊？ \- V2EX](https://v2ex.com/t/338091)
8. [安装xdpyinfo在Linux Xresources中设置正确的屏幕DPI\-教程\-Linux系统学习](http://8u.hn.cn/linuxjc/12241.html)
9. [HiDPI \- ArchWiki](https://wiki.archlinux.org/index.php/HiDPI#X_Server)
10. [Fixing HIDPI on a bare i3 install \(in Arch Linux btw\)](https://cyber.vodka/blog/hidpi-with-bare-i3-on-arch/)