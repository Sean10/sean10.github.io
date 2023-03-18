---
title: 全链路追踪APM初探
subtitle: APM
date: 2021-01-16 21:58:15
updated:
tags: [APM, Jaeger, OpenTracing]
categories: [专业]
---




订正: 因此唯一的解法就是避免打log的进程直接接触log文件了 -> 直接接触log文件的进程必须同时也是logrotate的进行者。

runit、s6、nosh 本来就已经把问题解决得很好了。


google搜索failed to get d-bus connection一类的问题一大堆



## 开机流程

### sysvinit

rc.local的使用习惯主要应该就是在这个版本的sysvinit中养成的了把。

### systemd

### dbus

#### native object


#### interface



## systemctl

编译systemd的花样又不一定了，又是一个什么叫做meson的东西，也不是cmake了的样子。njnja也不知道是什么东西。

反对各种抛开autotools体系抛开FHS抛开pkgconfig等等来自己起一套库依赖管理炉灶的方式。诚然xnix这套并不完美，但是这就是社区里的通用方式，通用语言，你不陪它玩，别人就没法愉快地陪你玩，需要迁就你的构建系统才能整合你的编译产出，而不是随手

meson目前来看，编译速度更快，并且默认就是携带着debug info的。


systemctl rescue主用内核，相比emergency功能稍全一些

systemctl emergency严重故障，如/etc/fstab坏了

### 诊断启动问题

使用如下内核参数引导
systemd.log_level=debug systemd.log_target=kmsg log_buf_len=1M

如果某个systemd服务的工作状况不含预计，希望调试的话，设置`SYSTEMD_LOG_LEVEL`环境比那辆为debu，

如在配置文件中加入
```
[Service]
Environment=SYSTEMD_LOG_LEVEL=debug
```

可以用--follow选项就检查日志

似乎直接通过journalctl -u xxx 搜集到的日志不如通过systemctl status获取到pid, 然后用journalctl -b _PID=123这种查到的全，有些运行时进程的日志元数据（例如_SYSTEMD_UNIT和_COMM)被乱序收集在/proc目录中。

也是可以使用内存转储的。



## docker使用

考虑到现在这个版本都是
rsyslog给出了几个buildbot，我还以为是在镜像内CI，似乎看起来并不是这样

buildbot好像是一个已经广泛使用了的CI和Jenkins结合使用的东西。





## automake编译

最后还是直接用automake编译了，但是现在才发现作者其实是写了个autogen.sh的脚本直接编译额的


另外似乎ssh上去的语言环境有点不正常，不知道和mac有没有关系，反正临时export修改一下看下。
export LC_ALL="en_US.UTF-8"

另外注意，我用的快捷键是IDEA的经典版本，而不是新版的样子，像Ctrl+G是行跳转之类的。



## postgresql 部署

brew install postgresql

 brew services start postgresql

initdb /usr/local/var/postgres
配置开机登陆（可选）：

$ mkdir -p ~/Library/LaunchAgents
$ cp /usr/local/Cellar/postgresql/9.3.5_1/homebrew.mxcl.postgresql.plist ~/Library/LaunchAgents/
$ launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
手动启动 postgresql

$ pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
查看状态：

pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log status
停止：

$ pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log stop -s -m fast

### kubeletes

brew  install minikube

<!-- minikube start -->
vmware驱动已经更新了
brew install docker-machine-driver-vmware

minikube start --vm-driver=vmware --registry-mirror=https://registry.docker-cn.com --image-mirror-country=cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --docker-env http_proxy=http://192.168.123.11:7890 --docker-env https_proxy=http://192.168.123.11:7890 --docker-env no_proxy=192.168.200.0/24


export http_proxy=http://192.168.123.11:7890;export https_proxy=http://192.168.123.11:7890; 
还是得配置代理，否则速度差太多了、

export http_proxy命令是添加命令行代理，主要是为了让minikube可以在命令行通过proxy去下载相关文件
--docker-env http_proxy...，设置虚拟机中docker daemon的环境变量，这里的环境变量http_proxy表示虚拟机中docker daemon使用的代理
--docker-env no_proxy，设置虚拟机中docker daemon不使用代理的地址段
--log_dir=tmp，设置minikube的日志存储位置，这里是当前目录下的tmp文件夹。该目录下会出现INFO和ERROR的日志，INFO是一定会有，ERROR是出错的时候才有。比如
--cpus 4，设置虚拟机的cpu核数
--memory 8192，设置虚拟机的内存大小，单位为M

作者：Mr_Hospital
链接：https://www.jianshu.com/p/48804c8bb250
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

M is unable to access k8s.gcr.io, you may need to configure a proxy or set --image-repository


minikube dashboard

kubectl run postgresql --image=postgres --env="POSTGRES_PASSWORD=password" --port=

 brew link --overwrite kubernetes-cli


 kubectl get pods -n kube-system

kubectl get all
这个可以看到ip的样子


  kubectl run postgresql --image=postgres --env="POSTGRES_PASSWORD=password" --port=5432 
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
Error from server (AlreadyExists): deployments.apps "postgresql" already exists


kubectl describe pod  postgresql-6bbf568dc9-vbw68

kubectl delete deployments.apps postgresql    


好像我操作错了，应该进入minikube ssh之后再操作


sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [ "https://dockerhub.azk8s.cn",
    "https://hub-mirror.c.163.com"]
}
EOF

原来minikube里面那个虚拟机直接设置代理，是可以通过
``` bash
https_proxy=<my proxy> minikube start --docker-env http_proxy=<my proxy> --docker-env https_proxy=<my proxy> --docker-env no_proxy=192.168.200.0/24
```
的--docker-env参数直接传递进去的

这样似乎能跑起来了。

kubectl get pods

的确，我把里面的配置改了，外面kubectl看到的就都是通的了
minikube service hello-minikube --url

kubectl delete services hello-minikube

要暴露出来，还得先exposed出来

kubectl expose deployment hello-minikube --type=NodePort --port=8080

minikube service hello-minikube --url

然后用minikube生成一个对外的链路

➜  k8s-docker-for-mac git:(master) minikube service postgres-test1 --url                               
http://172.16.2.130:32084


## 容器运行时？

minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=cri-o \
    --bootstrapper=kubeadm

这个是不是可以替代vm的作用？


➜  k8s-docker-for-mac git:(master) minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://172.16.2.130:2376"
export DOCKER_CERT_PATH="/Users/sean10/.minikube/certs"
# Run this command to configure your shell:
# eval $(minikube docker-env)

意思好像是从宿主机上直接使用vm里的Docker

的确，宿主机上的docker我关了，直接就可以看到里面的内容了。

kubectl似乎有一个上下文的概念，不同上下文中有不同的pods单元，是否代表是类似命名空间的一个层级？


#### kubectl create  minikube The connection to the server raw.githubusercontent.com was refused - did you specify the right host or port?

原来是因为我那个终端没开代理，导致无法访问github才报的……

➜  minikube_learn kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-kubernetes/master/all-in-one/jaeger-all-in-one-template.yml
^[[AError from server (AlreadyExists): services "jaeger-query" already exists
Error from server (AlreadyExists): services "jaeger-collector" already exists
Error from server (AlreadyExists): services "jaeger-agent" already exists
Error from server (AlreadyExists): services "zipkin" already exists
unable to recognize no matches for kind "Deployment" in version "extensions/v1beta1"




[Fix all\-in\-one start failed at k8s 1\.16 by boyoyon8 · Pull Request \#121 · jaegertracing/jaeger\-kubernetes](https://github.com/jaegertracing/jaeger-kubernetes/pull/121/commits/1dbd43ff185136665c4f9832ca581e31710b74d3)

参照上面这个pr修改后就能启动了。

[Use apps/v1 instead of extensions/v1beta1 \(required for k8s 1\.16\) · Issue \#2915 · projectcalico/calico](https://github.com/projectcalico/calico/issues/2915)

docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.16 --log-level=debug 是可以用的



怎么结合elastisearchne 
/*
这部分可能没用？
  To enable --es.tags-as-fields.all=true by environment variable its name should be ES_TAGS_AS_FIELDS_ALL.
*/
<!-- 
docker run -d  --name elasticsearch --restart=always -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" elastic/elasticsearch:latest -->

docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.6.0

[Install Elasticsearch with Docker \| Elasticsearch Reference \[7\.6\] \| Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)



r the location of Elasticsearch cluster, via --es.server-urls, depending on which storage is specified. To see all command line options run

go run ./cmd/collector/main.go -h

先关闭elasticsearch,再关闭collector，可能导致存储的数据索引异常，而无法查询uya


# use
docker run --rm --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -v /Users/sean10/Code/docker-images-tar/es.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /Users/sean10/Code/docker-images-tar/data:/var/lib/elasticsearch -v /Users/sean10/Code/docker-images-tar/log:/var/log/elasticsearch elastic/elasticsearch:7.6.0

    0     1     1     1 ?           -1 Ssl   1000   0:22 /usr/share/elasticsearch/jdk/bin/java -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dio.netty.allocator.numDirectArenas=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.locale.providers=COMPAT -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.io.tmpdir=/tmp/elasticsearch-17062322856269375926 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m -Des.cgroups.hierarchy.override=/ -XX:MaxDirectMemorySize=536870912 -Des.path.home=/usr/share/elasticsearch -Des.path.conf=/usr/share/elasticsearch/config -Des.distribution.flavor=default -Des.distribution.type=docker -Des.bundled_jdk=true -cp /usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch -Ediscovery.type=single-node
    1   110     1     1 ?           -1 Sl    1000   0:00 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

    容器内的执行方式

docker exec -it elasticsearch /bin/bash

-v /Users/sean10/Code/docker-images-tar/es.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /Users/sean10/Code/docker-images-tar/data:/var/lib/elasticsearch -v /Users/sean10/Code/docker-images-tar/log:/var/log/elasticsearch

chown -R 1000:1000 /Users/sean10/Code/docker-images-tar/log
chown -R 1000:1000 /Users/sean10/Code/docker-images-tar/data
chmod 777 /Users/sean10/Code/docker-images-tar/data
chmod 777 /Users/sean10/Code/docker-images-tar/log

``` yaml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch

```
哦，需要collector配置一下storage的url


<!-- 
docker run -d --link elasticsearch\
  -e SPAN_STORAGE_TYPE=elasticsearch \
  -e ES_SERVER_URLS=http://localhost:9200 \
  jaegertracing/jaeger-collector:1.16


docker run -d --name jaeger-collector --link elasticsearch\
  -e SPAN_STORAGE_TYPE=elasticsearch \
  -e ES_SERVER_URLS=http://localhost:9200 \
  jaegertracing/jaeger-collector:1.16 -->


docker run -d --rm --name jaeger-collector --link elasticsearch \
  -p14267:14267/tcp \
  -p14250:14250/tcp \
  -e SPAN_STORAGE_TYPE=elasticsearch \
  -e ES_SERVER_URLS=http://172.17.0.2:9200 -e ES_TAGS_AS_FIELDS_ALL=true \
  jaegertracing/jaeger-collector:1.16

查看帮助文档

docker run -it --rm jaegertracing/jaeger-collector:1.16 -h

docker run \
  -e SPAN_STORAGE_TYPE=elasticsearch \
  jaegertracing/jaeger-collector:1.16 \
  --help

--es.server-urls=http://localhost:9200
--span-storage.type=elasticsearch

docker run -d\
  --rm --name jaeger-agent --link jaeger-collector\
  -p6831:6831/udp \
  -p6832:6832/udp \
  -p5778:5778/tcp \
  -p5775:5775/udp \
  jaegertracing/jaeger-agent:1.16 --reporter.tchannel.host-port=172.17.0.3:14267 --log-level=debug 

socat端口转发
socat -d -d -lf ./socat.log UDP4-LISTEN:10086,bind=192.168.123.11,reuseaddr,fork UDP4:127.0.0.1:6831  
转发了6831端口

ncat -4u 192.168.123.11 6831
可以看到端口的确可以访问了


scope是openTracing 2.0引入的内容，

结果就只是最后close前没有sleep引起的……卧槽

start_active_span会默认继承上一个active的span形成链式。

最后找到了，少了sleep(2)这行，导致udp请求一直没发送出来。


[C\+\+ opentracing zipkin \- HEIS老妖 \- 博客园](https://www.cnblogs.com/resibe-3/p/8628561.html)

这个理论上应该是可以工作的才对。但是实际udp包还是没有发出来

jaeger的lobby这个库是用来干什么呢？

[blog/Jaeger源码分析—agent\.md at master · jukylin/blog](https://github.com/jukylin/blog/blob/master/Jaeger%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E2%80%94agent.md)


docker run -d -p 9100:9100 \
  -v "/proc:/host/proc:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/:/rootfs:ro" \
  --net="host" \
  quay.io/prometheus/node-exporter \
    -collector.procfs /host/proc \
    -collector.sysfs /host/sys \
    -collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"


 docker run --name prometheus -d -p 9090:9090 -v /Users/sean10/Code/learn_prometheus/prometheus/:/prometheus-data quay.io/prometheus/prometheus  --config.file=/prometheus-data/config.yml


Prometheus 的存储层在历史以来都展现出卓越的性能，单一服务器就能够以每秒数百万个时间序列的速度摄入多达一百万个样本，同时只占用了很少的磁盘空间。

[Prometheus核心开发者：从零开始写一个时序数据库 \- 更多 \- dbaplus社群：围绕Data、Blockchain、AiOps的企业级专业社群。技术大咖、原创干货，每天精品原创文章推送，每周线上技术分享，每月线下技术沙龙。](https://dbaplus.cn/news-160-2654-1.html)


[全链路监控（一）：方案概述与比较 \- 掘金](https://juejin.im/post/5a7a9e0af265da4e914b46f1)


docker save jaegertracing/example-hotrod:latest | gzip > example-hotrod.tar.gz   

docker转移images

docker load < busybox.tar.gz

移植centos环境中，封账成C库以供我们代码调用。

centos使用gcc高版本。

## cmake编译

这里的openssl可能需要由重辛提供一个高版本
yum install libcurl-devel zlib-devel
yum install openssl-devel
从零编译
wget 
./bootstrap --system-curl  --prefix=/usr/local/cmake
make
make install



 yum install centos-release-scl -y

 yum install devtoolset-9-gcc devtoolset-9-gcc-c++
  scl enable devtoolset-9 bash

  yum install llvm-toolset-7-cmake
  yum install llvm-toolset-7
  scl enable llvm-toolset-7 bash



  cmake -DCMAKE_INSTALL_PREFIX=$HOME/jaeger-cpp-client ..

docker说默认是网桥，为什么是172.17.0.1呢？然后这个ip为什么在这个里没有找到呢？

docker network inspect 30d50333d65676b6566ebb010ffd46092050819b8141fe63df5a9094204754f2

z[
    {
        "Name": "bridge",
        "Id": "30d50333d65676b6566ebb010ffd46092050819b8141fe63df5a9094204754f2",
        "Created": "2020-02-19T16:25:59.524900473Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "18a01dfb5c196a3c1686711797571bb31df203410b3227c31bc71858b63f70cf": {
                "Name": "elasticsearch",
                "EndpointID": "93e230d5b53880c446bc9a26db853e9b7342399f573f0158827cbd61a0d147cb",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "4337fc82bc65c52cd9749de57b6a19d08b9b6e05f56e7a610d6664f09485f4da": {
                "Name": "kibana",
                "EndpointID": "eca461f1e9affd293294158ab451f34bc0d05234d3d6ddd202f588acd83c1535",
                "MacAddress": "02:42:ac:11:00:06",
                "IPv4Address": "172.17.0.6/16",
                "IPv6Address": ""
            },
            "625678c6cc8288a5350bc4a8aa3fa12db9489a50fa789b8110d66439b022107f": {
                "Name": "recursing_payne",
                "EndpointID": "a522b2086516bb446ad2de8bb370c55744cd4299757ecdfb827970f6a6207192",
                "MacAddress": "02:42:ac:11:00:05",
                "IPv4Address": "172.17.0.5/16",
                "IPv6Address": ""
            },
            "7bddda694e8b86918cda06370dec7f218216bcb6907c068712653e23df39d736": {
                "Name": "jaeger-agent",
                "EndpointID": "de7bd59fe6dd0ca720e28475acd09773df7f0178869df97eabf10fb97d8b32cb",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "8b184d877b9f5d03b4049b4fccef7f49719f537ddaf3128c1dbd53bef150bad5": {
                "Name": "jaeger-collector",
                "EndpointID": "e1a9feb8d9659cebe1aa558c2b73df01db971e9c80fe8b243e0f3dc3c1caf30d",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "8bd8bd230cdf3cc52706a179877e8bc2ada34b1e2366330606efd04876d88f5e": {
                "Name": "friendly_hopper",
                "EndpointID": "1cddcb0036f160a3fd2863532a56f49497453a5b9831c319076c57c70cda6e2b",
                "MacAddress": "02:42:ac:11:00:09",
                "IPv4Address": "172.17.0.9/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]


这样看到的输出也还是怪怪的，怎么是172网段的呢？


ali也有在用jaeger

[对接Jaeger \- 查询与分析\| 阿里云](https://www.alibabacloud.com/help/zh/doc-detail/68035.htm)


cAdvisor用来收集宿主机上docker信息
 docker run -v /:/rootfs:ro -v /var/run:/var/run:rw -v /sys:/sys:ro -v /var/lib/docker:/var/lib/docker:ro -p 8080:8080 --detach=true --name=cadvisor --net=host google/cadvisor

docker run -d --name=grafana -p 3000:3000 grafana/grafana
默认密码admin

Could not create collector proxy","error":"at least one collector hostPort address is required when resolver is not available"
这里的resolver指的是什么？infra那个东西？

44.61:2811



docker run \
 --rm --name kibana \
  -p 5601:5601 \
  --link elasticsearch:elasticsearch_alias \
  -e "ELASTICSEARCH_URL=http://172.17.0.2:9200" \
elastic/kibana:7.6.0


docker inspect elasticsearch

docker run -d --rm \
  -p 16686:16686 \
  -p 16687:16687 \
  -e SPAN_STORAGE_TYPE=elasticsearch \
  -e ES_SERVER_URLS=http://172.17.0.2:9200 \
  jaegertracing/jaeger-query:1.16


[Tracers](https://opentracing.io/guides/python/tracers/)
propagator使用文档为啥始终注册不成功呢？

目前发现的问题是根本没有发送给agent，而让with先执行的情况下，就变成了invalid parent span id

尝试thrift工具抓包二次验证

thrift-tool --iface lo0 --port 6831 dump --show-all --pretty

6831端口没有抓到


docker run \
  --rm \
  --link jaeger-agent \
  --env JAEGER_AGENT_HOST=172.17.0.4 \
  --env JAEGER_AGENT_PORT=6831 \
  -p8080-8083:8080-8083 \
  jaegertracing/example-hotrod:latest \
  all

  配置一个官方样例吧……

https://cr.console.aliyun.com/?spm=a2c4e.11153940.0.0.52027e29lpnb0i

从这里获取阿里云的docker镜像源

好吧，配置了aliyun源之后，慢的只有elasticsearch这个镜像……

https://uj3vpkxi.mirror.aliyuncs.com

这个是我的源

docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elastic/elasticsearch:7.6.0 

### mac qq居然会占用大量网络，导致我的rdp卡的无法使用


### mac下coredump文件在哪

gdb调试方式

ll /cores/core.95645 
-r--------  1 sean10  admin   1.4G Feb 25 16:50 /cores/core.95645

cmake不知道增加这个会不会增加上debug信息

set(CMAKE_BUILD_TYPE "Debug")
 set(CMAKE_BUILD_TYPE "Release")

SET(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")



       SET(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")

cmake  -DCMAKE_BUILD_TYPE=Debug ..


hunter似乎有cache，当已经编译过的时候，可能可以通过关闭连接缓存服务器的配置让他可以编译

https://github.com/hunter-packages

hunter的创作者有提供HUNTER_DOWNLOAD_SERVER的配置选项，然后似乎本地源的配置也是有讲究的，我要找找有没有现成的自动下载指定源到本地的。

发现jaeger-client-cpp我折腾好像价值不大，这个hunter库里面依赖的版本挺多很老的，像是opentracing这种库，都是3、4年前的了，我直接用opentracing最新的吧。

不行，jaeger-client-cpp实现了很多网络请求

[Document library build/install and basic usage · ringerc/jaeger\-cpp\-client@2bb946b](https://github.com/ringerc/jaeger-cpp-client/commit/2bb946b75e4647b193084d2524dbe0949b05a293?short_path=04c6e90#diff-04c6e90faac2675aa89e2176d2eec7d8)

这个commit提交了很多文档，nice

cmake -DCMAKE_INSTALL_PREFIX=$HOME/jaeger-cpp-client ../jaeger-cpp-client
 -DHUNTER_ENABLED=0

 我仿佛从这个社区的帖子里看到了官方无力支持，导致用爱贡献的人没有心情和能力再继续下去了。

噢，和jaeger同级的其他都是收费服务，datadog和lightstep
[Evolving Distributed Tracing at Uber Engineering \| Uber Engineering Blog](https://eng.uber.com/distributed-tracing/)


uber落实zipkin为啥会自研的原因。

[一文讀懂微服務監控之分散式追蹤 \- IT閱讀](https://www.itread01.com/content/1565216403.html)

jaeger的某个使用？

[OpenTracing \- Jaeger 接 Elastic Stack \| 亂馬客 \- Re:從零開始的軟體開發生活](https://rainmakerho.github.io/2019/04/02/2019010/)

[Distributed Tracing, OpenTracing and Elastic APM \| Elastic Blog](https://www.elastic.co/cn/blog/distributed-tracing-opentracing-and-elastic-apm)

cmake的标准流程，不说明的话，基本就是
mkdir build
cd build
cmake ..
make
make install 了

像nlohmann这种纯头文件的编译，我看到的好像只是单元测试

那么cmake为什么必须要导入这个的xxx.cmake文件呢？不明白，required config的影响？


makeinstall的时候主要干了这个事情……
- Installing: /usr/local/lib64/cmake/nlohmann_json/nlohmann_jsonConfig.cmake
-- Installing: /usr/local/lib64/cmake/nlohmann_json/nlohmann_jsonConfigVersion.cmake
-- Installing: /usr/local/lib64/cmake/nlohmann_json/nlohmann_jsonTargets.cmake



cmake -DHUNTER_ENABLED=0  ../
-D
-DCMAKE_BUILD_TYPE=Debug
-DBOOST_ROOT=/opt/rh/rh-mongodb36


yum install centos-release-scl
#yum install rh-mongodb36-boost-devel.x86_64
#yum install rh-mongodb36-boost-static
yum install libevent-devel
yum -y install  openssl-devel
#yum install rh-mongodb36-yaml-cpp-devel

yum install devtoolset-7-gcc devtoolset-7-gcc-c++
yum install libcurl-devel

现在编译成功之后
export PATH=/usr/local/cmake/bin:$PATH

make -n 可以查看执行的命令，但是cmake这里好像没有直接调g++的

docker build -f Dockerfile.plugin --network host --build-arg http_proxy=http://192.168.123.11:7890 --build-arg https_proxy=https://192.168.123.11:7890 --no-cache  .

再构建一个docker版本的

我改了cmake文件，去掉CONFIG那项可以了，然后需要安装flex、bison


BUILD_SHARED_LIBS=ON

-DCMAKE_INSTALL_PREFIX=/usr/local/boost/

静态编译jaegertracing似乎-DBUILD_SHARED_LIBS=off增加这个就可以？、

scl软件收集

[3\.2 Release Notes Red Hat Software Collections 3 \| Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html-single/3.2_release_notes/index)

BOOST_LIBRARYDIR
``` bash

scl enable rh-mongodb36 bash


-DCMAKE_INSTALL_PREFIX=/usr/local/jaeger-client-cpp

cmake我指定了直接依赖的jaegertracing，理论上安装过了，怎么还提示找不到thrift的cmake文件呢？

按理来说，那个已经属于是安装完毕了的才对，不需要再去读cmake文件吧。

``` docker-compose
version: '3.7'
services:
 
  agent:
    image: jaegertracing/jaeger-agent:1.10
    command: ["--reporter.tchannel.host-port=collector:14267"]
  
  collector:
    image: jaegertracing/jaeger-collector:1.10
    environment:
      SPAN_STORAGE_TYPE: elasticsearch
      ES_SERVER_URLS: "${ES_SERVER_URLS}"
      ES_PASSWORD: "${ES_PASSWORD}"
      ES_USERNAME: "${ES_USERNAME}"
 
  query:
    image: jaegertracing/jaeger-query:1.10
    environment:
      SPAN_STORAGE_TYPE: elasticsearch
      ES_SERVER_URLS: "${ES_SERVER_URLS}"
      ES_PASSWORD: "${ES_PASSWORD}"
      ES_USERNAME: "${ES_USERNAME}"
    ports:
      - 8080:16686 # we will use 8080 to access UI

```
[全链路跟踪系统\(二\):实践篇 \- 钟华的博客 \| Zhongfox Blog](https://imfox.io/2017/11/19/hunter-2/)

某家公司的注入实战

[Go微服务全链路跟踪详解 \|](https://jfeng45.github.io/posts/go_opentracing/)


[Cross\-Process Tracing, 跨进程追踪 · opentracing文档中文版 \( 翻译 \) 吴晟](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/api/cross-process-tracing.html)

[分布式跟踪系统Jaeger\(二\)：Jaeger的基本概念 \- 青蛙小白](https://blog.frognew.com/2017/11/opentracing-jaeger-2.html)

[OpenTracing语义标准 \| opentracing\-specification\-zh](https://opentracing-contrib.github.io/opentracing-specification-zh/specification.html)

[OpenTelemetry Beta Release Plan \- OpenTelemetry \- Medium](https://medium.com/opentelemetry/opentelemetry-beta-release-plan-b046fbe0702b)

[opentracing\-cpp\-python/bindings\.cpp at master · isaachier/opentracing\-cpp\-python](https://github.com/isaachier/opentracing-cpp-python/blob/master/src/bindings.cpp)

tag适合的场景：

标识执行的关键执行路径，例如if else，tag可以标识到底走了那个路径
标识关键变量的值
标识error的发生和error msg
log适合的场景：

标识执行的时间，log会默认记录log生成的时间并按序排列，很适合标识函数内部执行时各个语句运行的时间
生成人为标记的特定数据，例如log请求参数

 wget http://archive.apache.org/dist/thrift/0.9.2/thrift-0.9.2.tar.gz \
     http://archive.apache.org/dist/thrift/0.9.2/thrift-0.9.2.tar.gz.asc

# now check the gpg signature please! Done? OK:

```
tar xf thrift-0.9.2.tar.gz
cd thrift-0.9.2
./configure --with-cpp --with-java=no --with-python=no --with-lua=no --with-perl=no --enable-shared=yes --enable-static=yes --enable-tutorial=no --with-qt4=no --prefix=/usr/local
make
make install
```


https://github.com/opentracing/opentracing-c/issues/18


vcpkg好像是微软提供的，等同hunter作用？

升级GCC版本可能带来的问题
如果升级的GCC版本在5.1以下，还是使用老版本的ABI，正常情况下，不会有ABI兼容性问题。

如果升级的GCC版本在5.1以上，使用c++11标准编绎的时候，默认使用了新版本的ABI。则可能对于项目中依赖的第三方的库，存在使用上的兼容性问题。如果是动态链接，则在启动的时候，会提示找不到对应的函数实现，即是典型的undefined symbol问题。

原因分析：
假设：A项目使用了B库（假设是动态库），B库在生成的过程中，使用的是旧版本的GCC进行编绎，自然ABI就是老版本。如果A库使用了5.1以上的GCC，且使用C++11标准的默认编绎，且使用的B库接口中含有string类型的参数。则A库在运行过程中，会提交对应的函数定义，即是undefined symbol。

原因是A的代码中需要寻找找的是新版本的ABI，而B库中生成的是老版本的ABI，两者虽然函数原型定义一样，但在库交互的层面，会找不到。

解决方案
1、最简单暴力的做法，就是把A库项目全部使用老版本的ABI来编绎。
成本：就是A上依赖的很多库，必须全部使用旧版本的ABI来安装。（例如依赖了新版本的protobuf，安装的时候使用了C++11的5.1以后的版本，则需要修改其源码安装配置，重新使用旧版本ABI来安装，否则会出现undefined symbol问题）

2、尝试去找B库的源码，看是否能使用新版本的ABI来编绎解决。

3、如果找不到B库的源码，尝试找B库的新版本库，看是否有支持新版本ABI。

4、写一个中间代理层，C库。C库中包含B库的静态链接，C库暴露出来的函数接口，不含有string和list。C库编绎的时候，使用旧版本ABI去链接B库即可。A库在使用C库的时候，由于接口中并不含有string和list，则可以使用新ABI进行调用。
————————————————
版权声明：本文为CSDN博主「werflychen」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/dreamvyps/article/details/89179248





docker build -f Dockerfile.plugin --build-arg http_proxy=http://10.239.4.80:913 --build-arg https_proxy=http://10.239.4.80:913 .

emm，原来是要用这个的
### Jetbrains卡顿

```
-Xms1024m
-Xmx4096m
-XX:ReservedCodeCacheSize=1024m
-XX:+UseCompressedOops
```

vm options里按照上述设置就可以了。

jetbrains高分频的某种设置换算分辨率时，会出现输入延迟的现象。让同时渲染的内容减少就会好转不少。



crash的runq好像可以显示排队进度？


docker的overlay、vlan跨主机网络通信主要指的是跨主机的docker之间的通信，对于我做了端口映射的场景来说，docker直连就可以了。




在-fpatchable-function-entry=之前，clang已经有多种在function entry插入代码的方法了：

-fxray-instrument。XRay使用类似-finstrument-functions的方法trace，和Linux kernel类似，运行时修改代码
Azul Systems引入了PatchableFunction用于JIT。我引入"patchable-function-entry"时就复用了这个pass
IR feature: prologue data，在function entry后添加任意字节。用于function sanitizer
IR feature: prefix data，在function entry前添加任意字节。用于GHC TABLES_NEXT_TO_CODE。Info table放在entry code前。GHC的LLVM后端目前仍是年久失修状态

[从\-fpatchable\-function\-entry=N\[,M\]说起 \- 知乎](https://zhuanlan.zhihu.com/p/104683907)




# 理解

[第 2 章：结构化数据的价值 · OpenTelemetry 可观测性的未来](https://jimmysong.io/opentelemetry-obervability/the-value-of-structured-data.html)

这篇讲的挺好, 可观测性的分析价值. 

