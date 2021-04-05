---
title: dockerfile学习
date: 2018-03-17 20:06:44
updated:
tags: [Docker]
categories: [专业]
---

现在装的环境有点多了，一不留神就会改动了其他工具使用的环境，这种环境下学下docker可以保障运行环境的一致性。

<!--more-->

试着先给毕设用的环境写个dockerfile, 顺便提了个PR，希望能被合并吧。

在找错的过程中，看到好多人用docker run来后台执行命令，感觉好微妙，每次从镜像新建一个容器来执行命令，那旧的容器是继续保留吗？还是说生产环境的确就是这样做的。

可能因为我这算是学习环境，所以对Docker的变动都是在内部修改，并不完全是从dockerfile来的吧。

``` bash
docker build -t sean10:tensorflow .
```

在docker run时，最后的命令行参数是会和dockerfile中的RUN, CMD, ENTRYPOINT相关的。

RUN是在Build时运行的，先于CMD和ENTRYPOINT。Build完成了，RUN也运行完成后，再运行CMD或者ENTRYPOINT。

比如这个，最后的bash会覆盖CMD指定的命令
``` bash
docker run -p 7777:8888 -v /Users/sean10/Code/LSTM-Sentiment-Analysis:/LSTM-Sentiment-Analysis -it tensorflow/tensorflow:1.1.0-py3 bash
```

------

# todo
* docker如何使用相对路径的文件内?   

# .dockerignore

>在docker命令行界面中发送上下文到docker的守护进程之前，它会检查上下文目录根路径下名为.dockerignore的文件，如果这个文件存在，命令行界面会修改上下文，排除那些被.dockerignore中的模式匹配到的文件和目录。

# docker stage
如何显示之前下载过的`stage`用的image ?

# 显示已下载的image的dockerfile

# systemd的docker内启动
> docker run -tid --name test_1 --privileged=true centos:latest /usr/sbin/init
> docker exec -it test_1 /bin/bash

# 多重from image
[精简Docker镜像的几个方法 \- 那少年和狗 \- 博客园](https://www.cnblogs.com/dogecheng/p/11437413.html)

# 参考
1. [Dockerfile里指定执行命令用ENTRYPOING和用CMD有何不同？](https://segmentfault.com/q/1010000000417103)
2. [跟我一起学Docker——RUN、ENTRYPOINT与CMD](https://www.binss.me/blog/learn-docker-with-me-about-run-entrypoint-and-cmd/)
3. [Docker容器怎样更改容器内应用程序的配置文件？](https://www.zhihu.com/question/61836409)


