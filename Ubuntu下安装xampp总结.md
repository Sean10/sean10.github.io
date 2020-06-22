---
title: Ubuntu下安装xampp总结
tags: [app]
date: 2015-10-24 19:59:00
categories: 专业
---

初用linux，对于指令还不太熟悉，在安装xampp过程中，一部分情况和其他教程不太符合，摸索着弄了一下。

在一开始没有用wget下载，在图形界面下下载的。
然后移动入opt文件夹，这里一开始始终弄不到管理员权限，不安装其他软件的情况下好像图形界面下取得不到管理员权限。
我用的是
<pre class="brush:bash";>然后用cd进入该文件夹</pre>
对文件权限设置可执行
<pre class="brush:bash";>chmod +x 文件名</pre>
然后启动文件
<pre class="brush:bash";>./opt/文件名  </pre>
即可