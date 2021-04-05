---
title: mac远程桌面踩坑
subtitle: rdp
date: 2020-07-11 12:12:23
updated:
tags: [mac, rdp, vnc]
categories: [专业]
---

## 背景
最近要做痔疮手术, 术后有一段时间估计是还不能坐着的. 台式机就没有办法用上了, 但是笔记本的性能还是有点差,内存也不太够.


## 工具
主要有以下以及更多的访问方式

* vnc的一系统访问方式
  * Apple Remote Desktop
  * VNC等工具
  * 封装了vnc协议的类似`xrdp`的工具
* Jump Desktop


## 踩坑实验记录
### vnc直接修改分辨率
一开始我直接用的自带的`finder`里的`vnc`访问的, 效果还可以.不过因为笔记本只有13寸,访问`3840*2160`的台式机, 分辨率用系统配置的`display`最低也只能设计成拟合`1920*1080`的, 相比我一般设置成`2560*1440`拟合`1280*720`的笔记本的字实在是小太多了. 

据我所知, `windows`的`rdp`是可以做到让渲染在客户端设备执行的逻辑的, 这样就无所谓我目标设备分辨率到底是多少了.

然而,很可惜, 经过调查, `mac`端主要也就上面这几种工具. `apple remote desktop`主要也只是基于`vnc`进行的封装,而vnc的渲染主要是在服务端进行的, 客户端只是直接显示的效果. 对于基于这种协议的远程访问工具只能直接修改服务端设备的分辨率来适应客户端, `linux`端至少看到的一般的解决方式都是这样的. 原生设置能够控制的分辨率还是有限, 最后我用了`SwitchResX`来进行的分辨率修改, 修改成拟合`1440*900`在客户端上用的.

### 无显示器操作方案[^2]
偶然看到好像有人提到没有接显示器的`mac mini`, 有人试着远程连接直接修改分辨率. 看到针对没有显示器连接的时候, 分辨率也能够修改. 我就试验了下`SwitchResx`, 发现居然的确显示了一个`Virtual Desktop`,可以修改的分辨率基本上都是客户端的分辨率. 基本上达到我的目的了.

### 无显示器竖屏方案[^3]
``` bash
#displayplacer
➜  ~ displayplacer list
Persistent screen id: FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF
Contextual screen id: 1104977160
Type: MacBook built in screen
Resolution: 1440x900
Hertz: 60
Color Depth: 4
Scaling:on
Origin: (0,0) - main display
Rotation: 0 - rotate internal screen example (may crash computer, but will be rotated after rebooting): `displayplacer "id:FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF degree:90"`
Resolutions for rotation 0:
  mode 0: res:1x1 hz:60 color_depth:4
  mode 1: res:800x600 hz:60 color_depth:4
  mode 2: res:1024x768 hz:60 color_depth:4
  mode 3: res:1280x720 hz:60 color_depth:4
  mode 4: res:1280x1024 hz:60 color_depth:4
  mode 5: res:1440x900 hz:60 color_depth:4
  mode 6: res:1680x1050 hz:60 color_depth:4
  mode 7: res:1920x1080 hz:60 color_depth:4
  mode 8: res:2560x1440 hz:60 color_depth:4
  mode 9: res:2560x1600 hz:60 color_depth:4
  mode 10: res:3840x2160 hz:60 color_depth:4
  mode 11: res:1024x640 hz:60 color_depth:4 scaling:on
  mode 12: res:1152x720 hz:60 color_depth:4 scaling:on
  mode 13: res:1152x800 hz:60 color_depth:4 scaling:on
  mode 14: res:1440x900 hz:60 color_depth:4 scaling:on <-- current mode
  mode 15: res:1680x1050 hz:60 color_depth:4 scaling:on
  mode 16: res:1920x1200 hz:60 color_depth:4 scaling:on
  mode 17: res:1504x846 hz:60 color_depth:4 scaling:on
  mode 18: res:1920x1080 hz:60 color_depth:4 scaling:on
  mode 19: res:2048x1152 hz:60 color_depth:4 scaling:on
  mode 20: res:2304x1296 hz:60 color_depth:4 scaling:on
  mode 21: res:2560x1440 hz:60 color_depth:4 scaling:on
  mode 22: res:3008x1692 hz:60 color_depth:4 scaling:on
  mode 23: res:3360x1890 hz:60 color_depth:4 scaling:on
  mode 24: res:3840x2160 hz:60 color_depth:4 scaling:on
  mode 25: res:4096x2160 hz:60 color_depth:4 scaling:on
  mode 26: res:1280x1024 hz:60 color_depth:4

Execute the command below to set your screens to the current arrangement:

displayplacer "id:FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF res:1440x900 hz:60 color_depth:4 scaling:on origin:(0,0) degree:0"
```

```
#Example:
displayplacer "id:18173D22-3EC6-E735-EEB4-B003BF681F30+F466F621-B5FA-04A0-0800-CFA6C258DECD res:1440x900 scaling:on origin:(0,0) degree:0"

displayplacer "id:FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF res:1440x900 scaling:on origin:(0,0) degree:90"
```

#### fb-rotate
``` bash
 fb-rotate -l

```

## 方案
最后总结的方案主要就是下述两种了.
1. 基于`SwitchResX`直接修改服务端的分辨率
2. 拔掉服务端连接的显示器,这个时候用`SwitchResX`基本上就可以设置成客户端的分辨率了.

目前我主要用的就是第二套方案了.



## Reference
1. [Apple Remote Desktop 真是垃圾中的战斗机 \- V2EX](https://v2ex.com/t/649594)
2. [Force the resolution on a headless mac mini server \- Ask Different](https://apple.stackexchange.com/questions/116809/force-the-resolution-on-a-headless-mac-mini-server)
3. [jakehilborn/displayplacer: macOS command line utility to configure multi\-display resolutions and arrangements\. Essentially XRandR for macOS\.](https://github.com/jakehilborn/displayplacer)
4. [CdLbB/fb\-rotate: A Unix utility to rotate the display on any Mac and switch the primary display back and forth between displays\.](https://github.com/CdLbB/fb-rotate)