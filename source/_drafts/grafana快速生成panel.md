
---
title: grafana快速生成panel
tags:
  - grafana
  - prometheus
  - ceph
categories:
  - 专业
subtitle: tech
date: 2023-03-07 16:40:53
updated:
---



主要思路参考[Grabana：使用 Golang 或 Yaml 生成 Grafana Dashboard \| 云原生之路](https://guoxudong.io/post/grabana-create-grafana-dashboard/), 计划使用[K\-Phoen/grabana: User\-friendly Go library for building Grafana dashboards](https://github.com/K-Phoen/grabana) 或者[panodata/grafana\-client: Python client library for accessing the Grafana HTTP API](https://github.com/panodata/grafana-client) 这个库来实现这部分代码.



## grafana json model

### template
[JSON model \| Grafana documentation](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/view-dashboard-json-model/#templating)


目前复制的模板里, `current`不能删

### panels

[JSON model \| Grafana documentation](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/view-dashboard-json-model/#panels)

目前复制的模板里, gridpos不能删, 不然就不存在了...


#### row里包裹panels居然不是在row项的panels里嵌入, 而是在这个row项后面直接同级追加timeserils的panel项

不对, 理解有误. 这样追加的, 需要手动折叠一次, 才会被归入对应的row里

除非collapses是关的, 这样自动就会被嵌入进去?看下嵌入进去的json_model长啥样吧.


跟collapse是强相关的, 这里其实就是如果是折叠的, 那row的panels里嵌入timeserials, 但是如果是展开的, 那就panels得是同级的.

我靠, 这全靠总结规律啊.

### gridpos 

y轴坐标可以不用填, 自动能够填充, 还挺好. 主要是集合了row之后就不太好算了.

[Feature Request: auto Y for gridPos · Issue \#233 · grafana/grafonnet\-lib](https://github.com/grafana/grafonnet-lib/issues/233)





# faq
## prometheus metrics parse时发现高版本Counter会自带`_total`

[Can't create a counter with the \`\_total\` metric name suffix · Issue \#677 · prometheus/client\_python](https://github.com/prometheus/client_python/issues/677)

暂时先手动去除了, 应该是ceph-exporter版本偏低, 所以没有遵循新版本的规范.


