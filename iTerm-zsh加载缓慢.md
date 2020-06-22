---
title: iTerm & zsh加载缓慢
date: 2018-03-16 20:31:12
updated:
tags: [mac, iTerm]
categories: [专业]
---

最近iTerm每开一个tab都要等4、5秒的时间，按照ssd的加载速度来说这是本不应该发生的事情。

顶部栏是从python->ruby等等的变化，最后才到菜单栏。

猜想应该是导入的配置文件中导入了过多。

找了一下其他人遇到的问题，发现果然主要都是nvm造成的影响，把nvm的导入注释掉，就恢复到了1、2s加载速度。

不过，除此之外，iTerm默认是从系统的/usr/bin/login启动，启动时需要读取apple 系统日志(apple system log)，所以这里有两种解决方案。

1. 清除日志

```
sudo rm /private/var/log/asl/*.asl
```

2. 更换启动shell
在Iterm Perferences>Profile>General>Command 设置为/bin/zsh

# 参考资料
1. [iTerm 2、Terminal 启动加速](https://blog.jonslow.com/iterm2-launch-accelerate/)
