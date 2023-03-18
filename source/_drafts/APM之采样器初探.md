---
title: APM之采样器初探
subtitle: tech
date: 2022-10-03 16:59:08
updated:
tags: [APM, 全链路追踪, OpenTracing, OpenTelemetry]
categories: [专业]
---



### 采样

[docs\-cn/sdk\.md at 6c275fcdec3714bf91832878c5541f53736cda37 · open\-telemetry/docs\-cn](https://github.com/open-telemetry/docs-cn/blob/6c275fcdec3714bf91832878c5541f53736cda37/specification/trace/sdk.md#sampling)
[Review approach & specify algorithm for TraceIdRatioBasedSampler \(ProbabilitySampler\) · Issue \#1413 · open\-telemetry/opentelemetry\-specification](https://github.com/open-telemetry/opentelemetry-specification/issues/1413)



* 全采样
* 默认支持的按概率采样率
* 每多少个采样几个


这里提到的就是关键只采样有异常的, 不需要的数据就不采

伴鱼通过链式组合processor达成.

[调用链追踪系统在伴鱼：实践篇 \| 伴鱼技术团队](https://tech.ipalfish.com/blog/2021/03/04/implementing-tail-based-sampling/)
TODO:[TiKV 高性能追踪的实现解析 \- 掘金](https://juejin.cn/post/6935395310382350344)




## openTelemetry 采样器


目前开发了的采样器主要是[opentelemetry\-collector\-contrib/processor at main · open\-telemetry/opentelemetry\-collector\-contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor)这里的几种


### [尾部采样器 tailsamplingprocessor](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/tailsamplingprocessor)

[Opentelemetry 采样最佳实践 \- 知乎](https://zhuanlan.zhihu.com/p/523355937)

### [attributesprocessor](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/attributesprocessor)



### [似乎目前openTelemetry未实现]LTTB采样算法 (Downsampling Time Series for Visual Representation)

核心目的是保持原有的波形规律

#### 模拟页面
* [Flot \- Downsample Plugin](https://www.base.is/flot/)


#### 最大三角形三桶算法


1. 将首尾2个采样点 划分到2个桶中
2. 将第三个桶的点作为临时点(直接取该桶的点中的平均值的点), 然后迭代选择第二个桶中组成的三角形面积最大的预选点.
3. 按步骤2, 依次将第四个桶的点作为临时点, 继续选择第三个桶的预选点

由此得到.

根据这个逻辑来说, 其实是存在由于临时点选错, 导致整张图的采样逻辑错误的情况的. 看看论文中是怎么说场景的呢?



![](APM之采样器初探/APM之采样器初探_2022-10-05-17-43-52.png)


## QoS



### TODO:采样和QoS 差别?

