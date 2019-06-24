emm,黑苹果过程中明明提示graphic driver fail to load，结果还是成功进入了系统初始化界面。

所以只是因为显存太小导致的启动速度慢？

反正现在显存有点问题，如果之后独显来了，直接可以用的话倒是就好了。

主要就等网卡来了，来了之后就可以上网安装那些东西了。

emm, 还是在显卡上出了问题，现在判断还是可能需要装那个什么evergreen?

---


systemd的主要问题不是在于systemd，而是在于其作者的公信力不足的样子。

并且systemd借鉴于os x的样子。

>由于systemd项目合并了udev, logind等基础设施，以及Gnome/KDE积极与systemd集成，这给其它开源内核的桌面用户（以及Debian这样的多内核发行版）造成了困扰。

>systemd/pulseaudio/avahi现在已经成熟，还是非常好用的。

>除了管理 daemon 之外，还实现了 socket-activation 来支持按需加载服务。

接管了太多设施，如 syslog 被 systemd-journal 取代，crond 也被 systemd 的 timer 单元取代，udev 也准备集成到 systemd 中来，未来甚至还可能取代 /etc/fstab。尽管这些新的服务大部分都是独立于主进程的，但是还是有整个系统被红帽控制住的感觉

订正: 因此唯一的解法就是避免打log的进程直接接触log文件了 -> 直接接触log文件的进程必须同时也是logrotate的进行者。

runit、s6、nosh 本来就已经把问题解决得很好了。


google搜索failed to get d-bus connection一类的问题一大堆



在Fedora里面，network的控制不再是由network.service来控制，而是一个NetworkManager.service这个东西来控制了，看来又是替代了原生内容的了把？


## 开机流程

### sysvinit

rc.local的使用习惯主要应该就是在这个版本的sysvinit中养成的了把。

### systemd

### dbus

#### native object


#### interface



## systemctl

编译systemd的花样又不一定了，又是一个什么叫做meson的东西，也不是cmake了的样子。njnja也不知道是什么东西。

反对各种抛开autotools体系抛开FHS抛开pkgconfig等等来自己起一套库依赖管理炉灶的方式。诚然xnix这套并不完美，但是这就是社区里的通用方式，通用语言，你不陪它玩，别人就没法愉快地陪你玩，需要迁就你的构建系统才能整合你的编译产出，而不是随手

meson目前来看，编译速度更快，并且默认就是携带着debug info的。


systemctl rescue主用内核，相比emergency功能稍全一些

systemctl emergency严重故障，如/etc/fstab坏了

### 诊断启动问题

使用如下内核参数引导
systemd.log_level=debug systemd.log_target=kmsg log_buf_len=1M

如果某个systemd服务的工作状况不含预计，希望调试的话，设置`SYSTEMD_LOG_LEVEL`环境比那辆为debu，

如在配置文件中加入
```
[Service]
Environment=SYSTEMD_LOG_LEVEL=debug
```

可以用--follow选项就检查日志

似乎直接通过journalctl -u xxx 搜集到的日志不如通过systemctl status获取到pid, 然后用journalctl -b _PID=123这种查到的全，有些运行时进程的日志元数据（例如_SYSTEMD_UNIT和_COMM)被乱序收集在/proc目录中。

也是可以使用内存转储的。



## docker使用

考虑到现在这个版本都是
rsyslog给出了几个buildbot，我还以为是在镜像内CI，似乎看起来并不是这样

buildbot好像是一个已经广泛使用了的CI和Jenkins结合使用的东西。


## brew使用

没在文档里找到如何下显示brew update的进度，不然老是以为阻塞了，还好我想起来一般开源软件都用verbose，就试了一下，看了brew的写法也是相当不错的。

不过就是也只能显示到fetching的进度了，fetching快不快应该只能靠监控流量了。


brew底层用的还是Git,git现在通过改host还是不怎么能提速，虽然现在用不了proxychains4，但是似乎可以直接用`git config --global http.https://github.com.proxy socks5://127.0.0.1:1086`直接用代理，唉，早知道这个就没那么多事情了。


docker内如何使用代理呢

启动容器
通过docker run --net=host启动容器，使容器与 host 共享网络。

配置 .gitconfig
这里需要填 host 的 内网 ip，可以通过 ifconfig | grep "inet " | grep -v 127.0.0.1 查询，Mac 直接打开网络偏好也能看到内网 ip。

[http]
        proxy = socks5://172.20.162.149:1080
[https]
        proxy = socks5://172.20.162.149:1080
接下来可以愉快地 git clone 啦！

至此，我们完成了在容器内使用 host 代理的配置，如果配置 proxychains4 等工具，也是一样的道理。

## automake编译

最后还是直接用automake编译了，但是现在才发现作者其实是写了个autogen.sh的脚本直接编译额的


另外似乎ssh上去的语言环境有点不正常，不知道和mac有没有关系，反正临时export修改一下看下。
export LC_ALL="en_US.UTF-8"

另外注意，我用的快捷键是IDEA的经典版本，而不是新版的样子，像Ctrl+G是行跳转之类的。