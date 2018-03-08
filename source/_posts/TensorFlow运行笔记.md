---
title: TensorFlow运行笔记
date: 2018-02-24 13:24:58
updated: 2018-03-02 01:00:00
tags: [TensorFlow]
categories: [专业]
---

为了防止环境问题，所以使用了docker配置的TensorFlow环境，

```
docker pull tensorflow/tensorflow
docker run -it tensorflow/tensorflow bash
```

>Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA

一开始在使用tf.Session()时就报了这个错误。暂时没理会。

--------

但是对docker 不那么熟悉，不知道代码和数据应该存在哪个位置，在那还是决定代码通过volume挂载到docker内。

```
docker run -p 7777:8888 --name=tensorflow_sean10 -v /Users/sean10/Code/LSTM-Sentiment-Analysis:/LSTM-Sentiment-Analysis -it tensorflow/tensorflow  bash

这个版本tensorflow支持python3
docker run -p 7777:8888 --name=tensorflow_sean10_py3 -v /Users/sean10/Code/LSTM-Sentiment-Analysis:/LSTM-Sentiment-Analysis -it tensorflow/tensorflow:latest-py3  bash

```

然后又接触到了jypter,才逐渐明白这个做的不是ide的任务，有更多其他有意思的功能。

```
jupyter notebook --ip=0.0.0.0 --allow-root

想要再次获取
docker exec -it <docker_container_name> jupyter notebook list
```
然后在主机浏览器内访问`http://localhost:7777?token=xxxxxf`

然后运行代码时，显示已经使用了python3的内核，但实际运行弹出的错误来源始终是python2.7，查到的说是默认的tensorflow image只有python2.7安装了tensorflow包，需要pull python3-latest版本才行。试试吧,的确更换后就可以了……

# 参考资料
1. [本地访问远程服务器的Docker容器的Jupyter Notebook](http://yuenshome.cn/?p=4633)
2. [关于Docker目录挂载的总结](http://www.cnblogs.com/ivictor/p/4834864.html)
