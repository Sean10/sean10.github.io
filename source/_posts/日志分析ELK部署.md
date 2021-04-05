---
title: 日志分析ELK部署
subtitle: ELK
date: 2021-01-11 03:03:17
updated:
tags: [ELK, ceph]
categories: [专业]
---


# 部署
``` bash
docker pull sebp/elk
docker run -p 5601:5601 -p 9200:9200  -p 5044:5044 \
    -v /Users/sean10/Code/docker-volume/elk:/var/lib/elasticsearch \
    -v /Users/sean10/Code/Algorithm_code/elk/logstash.conf:/etc/logstash/conf.d/logstash.conf\
     --name elk sebp/elk
```

# Filebeat
日志数据采集器

* 与logstash直接配置本地文件的读取是否存在差别呢?

把`elk-logstash`的证书[^6], 和这个证书对应的域名加到hosts里解析, 成功导入了宿主机里的日志. 不过这种不修改`filter`的逻辑的日志, 只是把log的每行都记录成message了, 其实索引没利用起来的感觉.
``` yml
output:
  logstash:
    enabled: true
    hosts:
      - elk:5044
    timeout: 15
    ssl:
      certificate_authorities:
          - /etc/certs/logstash-beats.crt

filebeat:
  inputs:
      paths:
        - "/var/log/ceph/*.log"

```


``` bash
docker run --name filebeat  --user=root -v /Users/sean10/Code/ceph/ceph-14.2.9/src/out:/var/log/ceph -v /Users/sean10/Code/Algorithm_code/elk/logstash-beats.crt:/etc/certs/logstash-beats.crt -v /Users/sean10/Code/Algorithm_code/elk/filebeat.yml:/usr/share/filebeat/filebeat.yml -v /Users/sean10/Code/Algorithm_code/elk/hosts:/etc/hosts elastic/filebeat:7.10.1
```

# logstash

## 配置 
``` 
input
    file {
        path => ["/var/log/*.log", "/var/log/message"]
        type => "system"
        start_position => "beginning"
    }
}



filter {
#  if [type] == "cephlog" {
 grok {
 # https://github.com/ceph/ceph/blob/master/src/log/Entry.h
 match => { "message" => "%{TIMESTAMP_ISO8601:stamp}\s%{GREEDYDATA:msg}" }
 match => { "message" => "(?m)%{TIMESTAMP_ISO8601:stamp}\s%{NOTSPACE:thread}\s*%
{INT:prio}\s(%{WORD:subsys}|):?\s%{GREEDYDATA:msg}" }
 # https://github.com/ceph/ceph/blob/master/src/common/LogEntry.h
 match => { "message" => "%{TIMESTAMP_ISO8601:stamp}\s%{NOTSPACE:name}\s%
{NOTSPACE:who_type}\s%{NOTSPACE:who_addr}\s%{INT:seq}\s:\s%{PROG:channel}\s\[%
{WORD:prio}\]\s%{GREEDYDATA:msg}" }
 }
 date { match => [ "stamp", "yyyy-MM-dd HH:mm:ss.SSSSSS", "ISO8601" ] }
#  }
}
```
从ceph文档里找来的Filter, 不过好像匹配失败了, 可能跟版本有关系吧.版本太低? 用第一个,至少时间戳出来了.

## 实验
``` bash
/opt/logstash/bin/logstash --path.data /tmp/logstash/data \
    -e 'input { stdin { } } output { elasticsearch { hosts => ["localhost"] } }'
```

## 导入log

似乎要导入log, 默认的`elk`的配置不行, 需要修改`logstash`的配置, 增加一些属性.

## 一般流程
> 在日志存储服务器上安装filebeat
> filebeat将log存储目录下的所有log全部读取，发送到kafka
> logstash从kafka上读取日志，格式化后发送到elasticsearch
> elasticsearch正确索引后，response给logstash
> logstash更新kafka的offset，读取新的记录
> 一直循环以上步骤，直到所有的log都被写入到elasticsearch当中

## 可能的问题
### 索引过多, 导致elasticsearch 429
> 正确的处理大量历史数据的导入
> 不过很诧异，我很难在网络上搜到有人提过429的问题。仔细回想以下，一定是我的集群太low了。整个elasticsearch cluster的索引处理能力太弱鸡。。。所以，在你的集群比较弱鸡的情况下该怎么处理？
> 
> 当然，最直接的方法就是减少索引的量，比如，你要导入10月1号之前的所有log，你直接把索引的名字命名为platform-2017-10-1，而不是platform-%{+YYYY.MM.dd}。就算是只有一个eleasticsearch node，2C4G，亦能在5分钟之内把1G的log全部索引完。等索引完这些历史数据之后，你再把logstash上的output规则改为platform-%{+YYYY.MM.dd}。

# elasticsearch
还是使用`volume`挂载
``` yml
path:
  data: /var/data/elasticsearch
  logs: /var/log/elasticsearch
```

# kibana 
导入是导入成功了, 基本的搜索也有了, 但是显示出来的冗余内容有点多.

# Reference
1. [spujadas/elk\-docker: Elasticsearch, Logstash, Kibana \(ELK\) Docker image](https://github.com/spujadas/elk-docker)
2. [elk\-docker](https://elk-docker.readthedocs.io/#usage)
3. [ELK生态：Logstash增量读取log文件数据，导入到Elasticsearch\_alan\_liuyue的博客\-CSDN博客](https://blog.csdn.net/alan_liuyue/article/details/92582101)
4. [用ELK导入历史log的正确姿势\_点火三周的专栏\-CSDN博客](https://blog.csdn.net/u013613428/article/details/78216068)
5. [手把手教你实战docker容器下的ELK环境搭建 \- 知乎](https://zhuanlan.zhihu.com/p/107346014)
6. [fileBeat和Elk整合的问题\_xinluke的专栏\-CSDN博客](https://blog.csdn.net/xinluke/article/details/52121906)