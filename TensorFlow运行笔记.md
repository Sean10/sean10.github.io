---
title: Docker TensorFlow运行笔记
date: 2018-02-24 13:24:58
updated: 2018-03-02 01:00:00
tags: [TensorFlow, docker]
categories: [专业]
---

为了防止环境问题，所以使用了docker配置的TensorFlow环境，

<!--more-->

为了防止万一被收录，论文查重问题，这篇之前一直没发布。

-------

# 部署

```
docker pull tensorflow/tensorflow
docker run -it tensorflow/tensorflow bash

发现这个仓库只能在1.1版本下跑，重新下载了个
docker pull tensorflow/tensorflow:1.1.0-py3
```

>Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA

一开始在使用tf.Session()时就报了这个错误。暂时没理会。

--------

但是对docker 不那么熟悉，不知道代码和数据应该存在哪个位置，在那还是决定代码通过volume挂载到docker内。

```
docker run -p 7777:8888 --name=tensorflow_sean10 -v /Users/sean10/Code/LSTM-Sentiment-Analysis:/LSTM-Sentiment-Analysis -it tensorflow/tensorflow  bash

这个版本tensorflow支持python3
docker run -p 7777:8888 --name=tensorflow_sean10_py3 -v /Users/sean10/Code/LSTM-Sentiment-Analysis:/LSTM-Sentiment-Analysis -it tensorflow/tensorflow:latest-py3  bash


docker ps -a -l
docker rm $container_id
docker rmi $image_id
docker stop tensorflow_sean10_1.1.0_py3
docker start -i tensorflow_sean10_1.1.0_py3

docker attach tensorflow_sean10_1.1.0_py3
docker exec -t -i tensorflow_sean10_1.1.0_py3 bash

docker run -p 7777:8888 -v /Users/sean10/Code/LSTM-Sentiment-Analysis:/LSTM-Sentiment-Analysis -it tensorflow/tensorflow:1.1.0-py3 bash

```

对运行中的docker进行网络映射
```
docker port tensorflow_sean10_1.1.0_py3
docker inspect tensorflow_sean10_1.1.0_py3 | grep IPAddress

pfctl
```

然后又接触到了jupyter,才逐渐明白这个做的不是ide的任务，有更多其他有意思的功能。

```
jupyter notebook --no-browser --ip=0.0.0.0 --allow-root

想要再次获取
docker exec -it <docker_container_name>
jupyter notebook list
```
然后在主机浏览器内访问`http://localhost:7777?token=xxxxxf`

然后运行代码时，显示已经使用了python3的内核，但实际运行弹出的错误来源始终是python2.7，查到的说是默认的tensorflow image只有python2.7安装了tensorflow包，需要pull python3-latest版本才行。的确更换后就可以了……

----------

jupyter默认没有配置自动wrap,

创建~/.jupyter/nbconfig/notebook.json配置文件，追加这一段
```
{
  "MarkdownCell": {
    "cm_config": {
      "lineWrapping": true
    }
  },
  "CodeCell": {
    "cm_config": {
      "lineWrapping": true
    }
  }
}

```

-----

`No module named ipykernel`

jupyter install 后 无法找到ipykernel_launcher

解决方法：
```
pip3 install pykernel
python -m ipykernel install --user
```

-----

# Tensorflow错误记录

## 输入维度错误

>ValueError: Input 0 of layer basic_lstm_cell_1 is incompatible with the layer: expected ndim=2, found ndim=1. Full shape received: [400]

输入数据占位符的维度错误了

>basicRNNCell ValueError: Shape must be rank 2 but is rank 1 for 'MatMul' (op: 'MatMul') with input shapes: [64], [64,32].

将BasicLSTMCell更换成BasicRNNCell时，始终出现错误。唉，在占位符embiding时没注意到传错数了，导致很久都没debug出来……

## dynamic_rnn

Dynamic_Rnn与Static_rnn主要区别在于输入数据的结构上。

>static_rnn 输入的list的大小[序列长度,batch_size,embed大小]，所以一一般在经过embed层后，使用x = tf.unstack(embed, seq_len, 1)变换为[序列长度,batch_size,embed大小]，然后输入到static_rnn，输出也是list，大小形状为[seq_len,batch_size,hidden_size],所以我们取最后一个的输出就直接为static_rnn_out[-1],但是我们的dynamic_rnn不是这样，它的输入是[batch,seq_len,input],所以一般经过embeding层后不需要变化直接输入dynamic_rnn里面，然后输出结果为dynamic_rnn_out，他的shape为[batch,seq_len,hidden_size] ,这样我们要取到最后一个的输出，需要多维度进行一个转化：tf.transpose(dynamic_rnn_out, [1, 0, 2])，然后才可以去计算loss和accuracy等。[5]
>
>tensorflow 的dynamic_rnn方法，我们用一个小例子来说明其用法，假设你的RNN的输入input是[2,20,128]，其中2是batch_size,20是文本最大长度，128是embedding_size，可以看出，有两个example，我们假设第二个文本长度只有13，剩下的7个是使用0-padding方法填充的。dynamic返回的是两个参数：outputs,last_states，其中outputs是[2,20,128]，也就是每一个迭代隐状态的输出,last_states是由(c,h)组成的tuple，均为[batch,128]。
>
>到这里并没有什么不同，但是dynamic有个参数：sequence_length，这个参数用来指定每个example的长度，比如上面的例子中，我们令 sequence_length为[20,13]，表示第一个example有效长度为20，第二个example有效长度为13，当我们传入这个参数的时候，对于第二个example，TensorFlow对于13以后的padding就不计算了，其last_states将重复第13步的last_states直至第20步，而outputs中超过13步的结果将会被置零。[6]

如果由于batch一次读入过大，可以使用API中的eval_in_batches(https://github.com/petewarden/tensorflow_makefile/blob/master/tensorflow/models/image/mnist/convolutional.py#L255)

# graph的权值偏置值复用[7]


如果不复用，训练的和测试的图就一点关系都没有了，就很有可能导致下面的那样输出结果完全无关

``` python3
tf.get_variable_scope().reuse_variables()
```


## lstm 准确率过低

输出的训练集上的loss还算正常，但是acc连50%都没有

根据知乎大神所说，二分类问题50%的准确率就等同于没有学习到，没有进行优化。
```
lSTM
loss:5.716831 batch_Acc: 0.5 acc:0.525
loss:0.16445649 batch_Acc: 0.9583333 acc:0.2225
loss:0.21239458 batch_Acc: 0.9583333 acc:0.2075
loss:0.07486567 batch_Acc: 1.0 acc:0.23
loss:0.015121219 batch_Acc: 1.0 acc:0.275
loss:0.10024478 batch_Acc: 0.9583333 acc:0.2475
loss:0.056348186 batch_Acc: 0.9583333 acc:0.25
loss:0.018785348 batch_Acc: 1.0 acc:0.255
loss:0.017489845 batch_Acc: 1.0 acc:0.255
loss:0.0008791888 batch_Acc: 1.0 acc:0.275
loss:0.0012423135 batch_Acc: 1.0 acc:0.27
loss:0.010488756 batch_Acc: 1.0 acc:0.2525
loss:0.008405162 batch_Acc: 1.0 acc:0.245
loss:0.0038538638 batch_Acc: 1.0 acc:0.2125
loss:0.00075430545 batch_Acc: 1.0 acc:0.2375
loss:0.0011128571 batch_Acc: 1.0 acc:0.2275
loss:0.014299699 batch_Acc: 1.0 acc:0.325
loss:0.00064932863 batch_Acc: 1.0 acc:0.3775
loss:0.0039427895 batch_Acc: 1.0 acc:0.425
loss:0.0043920926 batch_Acc: 1.0 acc:0.4025
loss:0.0037787214 batch_Acc: 1.0 acc:0.41
```


```
传统RNN
loss:4.7681885 acc:0.51
loss:1.3538479 acc:0.48
loss:1.7510027 acc:0.4825
loss:1.2965189 acc:0.5175
loss:0.8639291 acc:0.4725
loss:0.7651406 acc:0.48
loss:0.5035551 acc:0.4725
loss:0.7856639 acc:0.5125
loss:0.41615644 acc:0.535
loss:0.63352686 acc:0.515
loss:0.8058832 acc:0.5125
loss:0.715534 acc:0.5225
loss:0.58976775 acc:0.49
loss:0.6922045 acc:0.4875
loss:0.6963689 acc:0.5225
loss:0.6474119 acc:0.48
loss:0.6436865 acc:0.4725
loss:0.79238504 acc:0.4925
loss:0.63156253 acc:0.535
loss:0.6858673 acc:0.51
loss:0.65090233 acc:0.52
answer
```

模型暂时和原本的初版差别不大，训练集也还是原本的随机出的，测试集顺序理论上应该没有影响才对

就是上面那个weight和bias复用的问题……我定义在def model里了，导致reuse根本没有可以存下来

----

# tf.truncated_normal()
这个函数产生正太分布，均值和标准差自己设定。这是一个截断的产生正太分布的函数，就是说产生正太分布的值如果与均值的差值大于两倍的标准差，那就重新生成。

# bi-lstm实现

参考了[8][9][重点10]


# weight初始化导致无法梯度下降

使用Xavier initialization[11]


# basicRNNCell使用上需要在LSTMCell基础上修改一些地方

参考[13]的代码修改的，原来在rnncell的时候就直接取当前state计算就可以ile

# 使用MultiRNNCell的时候

>InvalidArgumentError: Dimensions must be equal, but are 128 and 464 for 'rnn/while/rnn/multi_rnn_cell/cell_0/basic_rnn_cell/MatMul_1' (op: 'MatMul') with input shapes: [24,128], [464,64].

要用python迭代器生成list，而不能直接用乘法，并且需要从函数来获取多个cell,不能直接对一个cell进行迭代

# tf.metric.recall使用前需要initilizer

```
sess.run(tf.local_variables_initializer())
```

使用完 发现 这个值依旧是不能用的，因为这个统计的是整个session周期内的

# 梯度消失和梯度爆炸，如何从指标上判断出来

梯度消失似乎主要指的是多层效果反而没有浅层的好

# 关于recall,f1等指标计算

按照[16]条执行，成功跑出来了。

# 使用IndRNN 报错，似乎和while_loop有关

>ValueError: Cannot use 'rnn/while/rnn/multi_rnn_cell/cell_0/ind_rnn_cell/Mul_1' as input to 'rnn/while/rnn/multi_rnn_cell/cell_0/ind_rnn_cell/clip_by_value' because they are in different while loops. See info log for more details.

看了下源码，出现了clip_by_value位置的只有设置了recurrent_abs_max的时候，所以把这个参数先取消看看，成功跑起来了~hhh

性能和普通的lstm差不多，之后再和官方的lstmcell比较一下源码，看看怎么改能够跑起来

发现问题，由于最初对tensorflow认识不足，导致训练集和测试集虽然复用了weight和bias，但是实际是运行在不同的网络中的，而IndRNN可能由于没能针对Tensorflow处理好这些问题，因此无法应用。在修改后的代码中，复用同一个网络后，这个参数添加后就没有报错了。

# 参考资料
1. [本地访问远程服务器的Docker容器的Jupyter Notebook](http://yuenshome.cn/?p=4633)
2. [关于Docker目录挂载的总结](http://www.cnblogs.com/ivictor/p/4834864.html)
3. [wrap jupyter config ](https://stackoverflow.com/questions/36419342/how-to-wrap-code-text-in-jupyter-notebooks)
4. [No module named ipykernel](https://github.com/jupyter/notebook/issues/1558)
5. [tensorflow中dynamic_rnn与static_rnn区别](https://blog.csdn.net/qq1483661204/article/details/78997644)
6. [tensorflow高阶教程:tf.dynamic_rnn](https://blog.csdn.net/u010223750/article/details/71079036)
# Reference
7. [Tensorflow 变量的共享](https://www.cnblogs.com/rocketfan/p/5773373.html)
8. [tensorflow bilstm官方示例](http://www.cnblogs.com/linyx/p/6979119.html)
9. [TensorFlow实战12：Bidirectional LSTM Classifier](https://blog.csdn.net/Felaim/article/details/70300362)
10. [tensorflow学习笔记(三十九) : 双向rnn (BiRNN)](https://blog.csdn.net/u012436149/article/details/71080601)
11. [谷歌工程师：聊一聊深度学习的weight initialization](https://www.leiphone.com/news/201703/3qMp45aQtbxTdzmK.html)
12. [ensorFlow教程：利用TensorFlow处理自己的数据](https://lguduy.github.io/2017/05/20/TensorFlow%E6%95%99%E7%A8%8B-%E5%A4%84%E7%90%86%E8%87%AA%E5%B7%B1%E7%9A%84%E6%95%B0%E6%8D%AE/)
13. [RNN入门：利用TF的API（二）](https://www.jianshu.com/p/8e96b7ee1421)
14. [A bug in tensorflow r1.4 when applying MultiRNNCell](https://github.com/tensorflow/tensorflow/issues/14897)
15. [Add precision and recall summaries ](https://github.com/dennybritz/cnn-text-classification-tf/issues/71)
16. [tensorflow中precision,recall和F1](https://blog.csdn.net/appleml/article/details/58647585)
17. []()
