---
title: kcp协议理解
date: 2018-03-23 14:24:38
updated:
tags: [kcp, tcp, 网络]
categories: [专业]
---

# TCP Qos 问题

TCP的流量控制仅针对单条TCP连接，而在网络中UDP、ICMP等包并不遵循这个规则，TCP迫于流量控制，检测到端到端阻塞就会减小自己的窗口，然后被UDP等占据闲置出的带宽，导致饿死。

由于这个TCP在阻塞情况下迫于大环境无法保证传输质量，就有大批转而使用UDP协议加速的方式。

<!--more-->

# UDP 加速

kcp的设计是为了解决在网络拥堵情况下tcp协议的网络速度慢的问题。(包括BBR等，还有很多)一般使用udp作为下层传输协议。

>KCP对TCP的一些细节进行了改良[1]
>
>* RTO翻倍VS不翻倍
>* 全部重传VS选择性重传
>* 快速重传
>* 非延迟ACK VS 延迟ACK
>* ACK+UNA VS UNA
>* 非退让流控
>	* KCP正常模式同TCP一样使用公平退让法则，即发送窗口大小由：发送缓存大小、接收   端剩余接收缓存大小、丢包退让及慢启动这四要素决定。但传送及时性要求很高的小   数据时，可选择通过配置跳过后两步，仅用前两项来控制发送频率。以牺牲部分公平   性及带宽利用率之代价，换取了开着BT都能流畅传输的效果。

## RTT(round trip time) && RTO(Retransmission timeout)

RTT: 数据包往返时间
RTO: 超时重传时间

RTO重传间隔是指数增加的。丢包一次后，下一次重传RTO会从1，2，4，8……。叫做指数回避策略。(RTO下限200ms,写在协议里的，无法修改)


# 数据链路层ARQ与传输层ARQ区别

数据链路层的ARQ只能保证点到点之间传输的可靠性。

如A-B-C，数据链路层只能保证A-B不丢包，但是B会不会由于传输窗口满而丢弃包就无法保证，A-C之间传输的可靠就需要上层传输层协议来确认，这就叫做端到端的可靠性。

# Reference
1. [kcp 详解](https://www.oschina.net/p/kcp)
2. [RTO 对TCP超时的影响](http://weakyon.com/2015/07/30/the-impact-fo-rto-to-tcp-timeout.html)
3. [TCP RTO 计算方法及Go实现验证](https://sheepbao.github.io/post/tcp_rto_research_and_golang_implement/)
