---
title: pdf解密脚本
date: 2018-01-11 10:36:16
tags: [shell, python, script]
categories: [专业]
---

这两天要打印pdf,结果发现老师的pdf被加密了，需要解密一下，就心血来潮想解决一下。

<!--more-->

一开始想到的方法是，pdf可以通过浏览器打开，那么其实也就完全可以通过浏览器截图，自动下拉继续截图生成新的pdf，这算是一种解决方法嘛。似乎是可以通过浏览器打印保存pdf也能实现解密，不过莫名的pdf expert直接就能编辑被加密过的pdf。所以没想到怎么判断是否被加密。

总之，回来之后发现用linux下的ghostscript工具，可以先通过pdf2ps\ps2pdf转换一下，转换回来之后就是无加密状态的pdf了。

但是看了看，需要转换十来个，每个需要转换的时间还挺长的，索性就看看能不能从python调用程序吧。

就有了这样一个版本的代码(github repo)[https://github.com/Sean10/Algorithm_code/tree/master/script/pdf_decrypt]。

python有2个模块可以做到调用外部程序需求，os.popen与基于前者之后升级后的subprocess.popen。

单线程执行虽然基本可以完成需求，但是多线程应该能快一些吧，如果这个模块是fork出一个子进程调用其他进行操作的话，而不是在python内部执行的话（GIL锁）。

现在每个语言基本都有了concurrent模块，对multithread和multiprocess进行了新的封装实现。

基本使用还是比较简单的，```pool = subprocess.threadPoolExecutor```

之后将函数添加进入就可以执行了,`pool.submit(func,para)`，就能实现出基本的调用操作。

不过，还遇到了一些问题，如，使用call而不是popen时，ghostscript打开文件总会报一些错误,换回popen就一切正常。

还有，subprocess和线程池进程池同时使用，效率并没有多大的提升，猜想哪个部分可能出现了阻塞？


--------------


之后，考虑到暂时不太能区分出差别的情况，决定用shell试试。

其中主要遇到的问题就是文件名如果存在空格，就会因满足IFS（the Internal Field Separator），导致文件名被分割，最终的解决方法是在获取文件名前修改IFS,运行结束时再恢复。

多线程部分因时间问题暂时没再写，不过可以参考该博客，（linux_shell如何实现多线程)[https://www.cnblogs.com/signjing/p/7074778.html].
