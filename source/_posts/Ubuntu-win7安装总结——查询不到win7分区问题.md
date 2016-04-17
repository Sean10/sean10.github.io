---
title: Ubuntu+win7安装总结——查询不到win7分区问题
tags: [app]
date: 2015-10-24 20:01:00
categories: 专业
---

这两天在学校计划装个linux以后用来写代码，windows下面就专心玩好了。
本来搜了下经验，以为很轻松（虽然事实上知道怎么弄，不出错就确实很轻松啦）。然后，就那么把原来的win7系统不自觉的全格了，直到花了1天半的时间之后，搜linux下怎么看不到win7的硬盘搜不到结果，忽然点开linux下的computer的property看了下之后，发现原本一直以为的linux只是在隔出来的50G下玩的，事实上是500G。
然后，明白了，数据全没了。（虽然已经备份，但是对于一些下载的计划看的资料丢了，也是比较难很快接受）。
OK，知道现在电脑里什么都没了以后，轻松了/

因为觉得在Linux下装win7，事后的引导什么的可能会很麻烦，就依旧先刻个win7的U盘，在win7安装之后安装ubuntu。
这里的win7是从msdn里下载的应该说是最纯净的win7，因为连个网卡驱动都没有，还好周边有一台老式机，下载了一个拷过来才用上的网络。
接下来就是唯一导致这次安装错误的幕后黑手。

windows下准备好ultraiso刻录好的ubuntu的U盘。
开机进入选择启动（lenovo是F12进入，我电脑已经设置为Legacybios，所以不太了解win8双系统uefi该怎么设置）
进入ubuntu安装界面以后，之前都按continue就可以，到分区前一步，如果显示出已检测到win7系统，那么就说明你硬盘格式和标识都是正确的，下一步也就不会出现下一步不显示win7已经分好的磁盘的问题了。

如果没有显示检测出win7，现在按照我查的资料来理解。
安装win7的时候是MBR分区模式，安装ubuntu时，自动识别为GPT分区模式，执行以下命令就可以让Ubuntu以MBR执行。
在试用版里输入一下命令即可
<pre class="brush:bash";>sudo dd if=/dev/zero of=/dev/sda bs=1 count=8 seek=512</pre>
这会抹去 Primary GPT header 里的 GPT signature。请不要输错任何一个字，包括空格。
有其他人说也可以在bios设置里设置Legacyonly.我电脑没有这个选项，不确定在什么上有那个。
以上。