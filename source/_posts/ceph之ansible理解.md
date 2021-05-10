---
title: ceph之ansible理解
subtitle: ansible
date: 2021-02-18 19:36:07
updated:
tags: [ansible, ceph, linux]
categories: [专业]
---

# 理解
大家最受影响的就是一半时间开发, 一半时间运维的这个问题. 利用一个成熟的配置文件, 快速部署起自己需要的环境, 而不是一步步通过页面控制点击, 这是一个很省力的事情. 将运维中与部署相关的事情通过一些成熟的模板化的配置文件来提供, 最好不过了.


## ansible比较大的几个优点
* 由于是开源框架, 基于ansible的针对其他开源软件的配套代价较多
  * 比如集成钉钉等
* 配置文件化管理部署成熟
  * 可针对不同的环境, 使用不同的配置文件快速部署
  * 可以使用开源插件快捷对接配置管理服务
    * 因为 Ansible2.0以上的版本已经原生集成了consule: consul_module

## 不建议的操作
建议遵循`unix`思想, 一个工具只做该做的事情, ansible虽然提供了能力, 但是能由脚本来完成的事情, 没必要在ansible中进行处理
根据[^10]终于意识到我现在所处理的最别扭的地方在哪里了...这篇文章中说了下述几点.

* 在 ansible 中使用复杂的语法规则
  * 更推荐直接封装成脚本, 由脚本来进行这些操作.但是脚本的粒度拆分, 什么应该让ansible来操作, 什么应该让脚本来处理, 是个边界.
* 明明几行 Shell 就可以搞定的事情，为什么一定要使用 Ansible 来做呢？
  * 明明一个 Shell 脚本就可以完成的环境监察，为什么一定要使用 Ansible Playbook 来做呢？要知道 Playbook 编写语法虽然是 YAML，但是使用起来并不简单，有很多特殊的语法需要去注意，完全没有必要花费精力去学习一个新的工具去完成。
 * 如果脚本中存在较多判断，不宜使用 Playbook 实现逻辑
 * 如果脚本中存在部分参数解析功能，不宜使用 Playbook 实现逻辑
 * 不要过度拆分 task，保证每个 task 完整性

> 从个人使用上来说，Ansible 还是很好用的，至少它无需 Agent，SSH 连接等特性，使用起来很友好。
> 
> 但是我们也不应该过分使用 Playbook，编写 Playbook 解析 json 花费了不少的时间，远不如直接在被执行脚本中完成的成本低。


## tidb开发计划

> deploy.yml 用来部署集群。执行 deploy 操作会自动将配置文件、binary 等分发到对应的节点上；如果是已经存在的集群，执行时会对比配置文件、binary 等信息，如果有变更就会覆盖原来的文件并且将原来的文件备份到 backup（默认） 目录。
> 
> start.yml 用来启动集群。注意：这个操作只是 start 集群，不会对配置等信息做任何更改
> 
> stop.yml 用来停止集群。与 start 一样，不会对配置等做任何修改。
> 
> rolling_update.yml 用来逐个更新集群内的节点。此操作会按 PD、TiKV、TiDB 的顺序以 1 并发对集群内的节点逐个做 stop → deploy → start 操作，其中对 PD 和 TiKV 操作时会先迁移掉 leader，将对集群的影响降到最低。一般用来对集群做配置更新或者升级。
> 
> rolling_update_monitor.yml 用来逐个更新监控组件，与 rolling_update.yml 功能一样，面向的组件有区别。
> 
> unsafe_cleanup.yml 用来清掉整个集群。这个操作会先将整个集群停掉服务，然后删除掉相关的目录，操作不可逆，需要谨慎。
> 
> unsafe_cleanup_data.yml 用来清理掉 PD、TiKV、TiDB 的数据。执行时会先将对应服务停止，然后再清理数据，操作不可逆，需要谨慎。这个操作不涉及监控。
> 

## 并行任务引擎

> 作业编排：可以自由选择相应的原子化操作，编排和发布新的作业流程
> 
> 可视化: 通过拖拽鼠标编排作业流程和参数表单，作业运行与任务的状态可视化
> 
> 并行调度: 同一个作业中的任务支持并行执行，在很多场景下能极大的缩短执行时间
> 
> 实时监控: 任务的运行状态在页面上实时更新
> 
> 断点执行: 执行错误的任务在修复错误后可以继续执行，不需要重新执行整个作业
> 
> 任务回滚: 遇到错误，可以支持回滚作业中已经执行过的任务，恢复执行环境
> 
> 作业模板: 可定制作业模板，用模板创建作业实例，避免重复工作
> 
> 任务组: 在作业模板中把任务分组，在创建任务实例时候可以动态的复制任务组，相似的作业使用同样的模板，但又不限制于模板
> 
> 计算表达式: 使用表达式动态的计算任务的参数值，能够极大的简化重复参数的输入以及复杂参数的拼接
> 
> 动态参数：前面任务的输出作为后续任务的输入参数

# 同类比较
* fabric
  * 快速入门使用
* puppet
  * C/S架构
  * 大部分付费功能
* Chef
* ansible
  * 语义多层架构
  * 基于ssh, push/pull均支持
  * cephadm是使用工具的配置语法进行部署后, 为了更强程度的自定义而进阶开发的产物.
* SaltStack
  * 基于python的开源CM工具
  * CS架构
  * 学习曲线低
  * cephadm之后有配套saltstack的
    * [ceph/ceph\-salt: Deploy Ceph clusters using cephadm](https://github.com/ceph/ceph-salt)
* Kolla-ansible?
  * 应该是openstack用的
  * 苏宁的产品化Hull思路
    * 基于上面的, 设计标准的RESTAPI
    * 定义Workflow
    * 可视化安装
    * 增加Data Center, cluster, region等逻辑概念, 更好的满足用户的部署需求.




## 最佳实践

### 重试逻辑 `--start-at-task`
这里采用的方案是自定义一个`callback`的插件, 将`failed`的`task`记录下来,见[callback](#callback), 然后封装一个`CLI`, 在下次执行的时候增加`--start-at-task {task_name}`来进行调度.
### 优化方案
和一开始调研时的目的有点不一样, 现在发现的主要问题在于ansible做的预处理的逻辑太多了, 而且主要框架严重依赖单独的ssd和shell, 这样带来的问题你就是部署的性能较差.

需要找一些优化的方案

* facts收集
  * 关闭
  * 设置缓存
* 增加并发
* 修改运行策略为free等.
* ssd长连接
* 开启pipelining.
  * 原本的逻辑是将library,role等生成一个脚本, 复制到远程主机上, 再执行. 现在改为通过pipe直接传递给ssh会话.
* 开启accelerate模式, 是需要对面的远程服务, 预计是通过rpc类似?
* 通过poll设置一段时间后再来查询, 不阻塞着等待了.

## 插件开发
### callback
主要目的是支持可以解析出当前失败的任务, 然后给另一个进行重试的接口进行使用.

## rolling upgrade什么意思

好像应该叫做rolling update模型, 哦,滚动升级

哦, 忽然明白为什么是loop结合delegate_to了, 因为要的不是并发,而是滚动操作.



#### 继承的方法
* v2_runner_on_failed
* runner_on_failed

这俩函数有什么区别呢? 目前看起来v2版本并没有加强多少? 那就随意使用了先.


## 理解开发方式
感觉[^3]这种插件的阅读方式能提供一些参考.

### python调用ansible

[python3\.8 调用ansible 2\.9\.4 api \| 经验分享&服务器代维](http://blog.65535.fun/article/2020/7/9/100.html)


# ansible使用

## 调试方式
* epdb
* -vvvvvv 
  * 虽然文档中只写了`-vvv`, 但是实际上代码里有一部分写了`vvvvvv`的函数.
* 设置ANSIBLE_KEEP_REMOTTE_FILES=1, 然后再运行Ansbile命令
  * ansible运行前生成的临时脚本就会保存下来, 不会被删除了.

## tower付费部分

* workflow?
  * workflow和任务队列的区别应该在于低代码吧?
  * 还是说图形界面的任务的图形化操纵叫做workflow?


## 练习方式
[chusiang/ansible\-jupyter\.dockerfile: Building the Docker image with Ansible and Jupyter\.](https://github.com/chusiang/ansible-jupyter.dockerfile)

[Ansible 用 Docker Compose 练习 Ansible\_w3cschool](https://www.w3cschool.cn/automate_with_ansible/automate_with_ansible-7wir27p7.html)



## 执行
通过`playbook`调度`role`的`task`, 来进行批量任务执行.
### workflow操作多个playbook是否有开源替代方案?
还是直接写playbook, 通过role控制. 目前来看, 不算特别关键.


## 语法
### playbook
``` bash
ansible-playbook -i {inventory host} add-osd.yml

# check
ansible-playbook foo.yml --check
```

### delegate_to
这里为什么看到和`loop`一起使用? `delegate_to`不支持直接指定组嘛?为什么要`loop`

这个的主要用途难道不是到一个只需要一个组内的一个节点执行的时候, 直接选中某一个吗?

ansible解析参数的过程, 和delegate_to这种指定某个节点只执行一次的逻辑, 是哪个优先呢?

如果是后者, 那是否这个的目的是用来让这个组内只在第一个节点执行一次?

但是感觉有点像是后者

``` yaml
delegate_to: "{{ item }}"
loop: "{{ xxx }}"

```

### run_once
当我不需要指定哪个节点来执行一个集群任务的时候, 直接用run_once就可以ile,他会直接指定第一台.

### role
自定义模块, 内部包含具体的`task`

#### 依赖管理

> 在 playbook 中存在多个 roles，且其中有相互依赖关系时，合理使用 meta 配置，填写其所依赖的 roles。注意，被依赖的 roles 会优先执行

#### include_task
在role内使用`include_task`无法使用`start-at-task`直接跳转, 而`import_task`可以

### ad-hoc
全节点临时执行的命令, 这个对于排查问题全节点查日志的时候还是用的比较多的.

``` bash
ansible 主机或组 -m command  -a '模块参数'  ansible参数
```


# ceph-ansile


[^2]里稍微解释了一部分.

## 问题
根据目前的调研结果来看, 存在两个问题
* osd层无法支持用于db,wal分区之后剩余空间创建osd
* crush rule无法自由指定
* 不支持tier创建
* 不支持针对ec模式下的资源池,通过创建副本的元数据资源池进行创建rbd.

### 二次开发问题
* 开发环境到底怎么才能部署起来呢?如果不使用实机已经装好ceph和ceph-ansible的环境, 就没有办法了?
* 为什么看vagrant.yml.sample里, 都是从一个裸机开始呢?怎么里面就不能装好ceph依赖吗?

姑且按照[^5]先启动看看.`

[osism/ceph\-ansible Tags \- Docker Hub](https://hub.docker.com/r/osism/ceph-ansible/tags)

vagrant 启动失败了, 似乎我用的centos景象不太一样, 找个docker hub里的试试

[Ceph — OSISM documentation](https://docs.osism.de/configuration/environments/ceph.html)

所以library其实完成的任务就是拼接command?

[AnsibleAPI 开发 \- 简书](https://www.jianshu.com/p/ec1e4d8438e9)
参考上面这篇文章基本可以说明怎么使用ansible提供的库


## 实践中的一些疑问
### command和 debug冲突?
对应的是不同的Task, 因此的确会冲突

### shell和command有没有区别?
一个是启动`shell`, 一个是启动`sh`
默认ansible使用的module 是 command，这个模块并不支持 shell 变量和管道等，若想使用shell 来执行模块，请使用-m 参数指定 shell 模块,但是值得注意的是普通的命令执行模块是通过python的ssh执行。


### changed 是用来标记task的幂等性满足与否?
的确, 用来标记这个任务是否对环境产生了修改?

### groups保留字
属于ansible自身的保留字段

### 变量和inventory还是有点区别的, 虽然是这个主机组的, 但是单纯的变量其实没有主机组的概念的.

### 为什么ceph没有提供给ansible基于rados的接口呢?
是否是因为使用的是ssh等协议, 所以用Rados其实并不太方便? 并没有设计这个connection?如果要开发, 其实是开发一个connection机制吧?

主要用途是部署, 而在部署阶段, rados得在mon部署完毕后才能工作, 因此如果要切换connection, 还必须分阶段. 还不如直接ssh全套


### You cannot register a variable, but you can set a fact (which in most use cases, including yours, will be equivalent to a variable):
``` yaml
- set_fact:
  the_output: "{{ restdata.json.parameter[1] }}"
```

### 要使用json_query , 需要安装jmespath
### ansible使用register就会被标记成changed?
### 为什么我用filter, 带上2个大括号就无法识别出对象了呢?
因为filter是jinja2语法, 外面需要先带上双引号. 
* You are using the debug: module with the option var: and this expects a variable, not a jinja2 template.

### 对于您的模块支持检查模式，您必须在实例化AnsibleModule对象时传递supports_check_mode = True。 当启用检查模式时，AnsibleModule.check_mode属性将计算为True。 
对task进行检查模式
### ansible的模块里打印大量内容, 即便他不显示,也会记录到warning里? 有垃圾产生?

### ansible的library不允许使用print, 居然就会报错. 没有日志, 需要之后再找怎么记录日志的方式
通过logging直接往其他文件, 或者通过Syslog协议等输出是一种方案.

或者可以记录到stderr中, ansible只检查stdout是否被污染, 然后报错.

不推荐用这个library, 将来自改造都不方便.

### out不能是变量? 只能是字符串?
这个还需要找一下

### 通过debug可以打印出Ansible_facts, 但是直接引用就一直打印不出来, 忽然脑子过了一下, 想通了, ansible_facts应该是上层概念, 对于使用的应该直接填key就可以了, 然后的确拿出来了.

### 字符串输出转换, 坑
  * `{}` 会把字符串转换成object, 导致即便to_json了, 双引号也没了, 其他程序无法识别了.
  * jinja2的语法问题
  * [Double quotes are converted to single quotes on variable assignment if contents look like JSON · Issue \#44897 · ansible/ansible](https://github.com/ansible/ansible/issues/44897)
  * [Ansible \- passing JSON string in environment to shell module \- Stack Overflow](https://stackoverflow.com/questions/41144922/ansible-passing-json-string-in-environment-to-shell-module)

### ansibleSequence是什么东西?


### 字典的key索引不会被Jinja2解析, 这个真的是深坑


* 感觉看了下, 这个坑很大啊...一点都不方便debug



![](ceph之ansible理解/ceph之ansible理解_2020-11-12-00-49-50.png)

### 莫名其妙的module failure
 没有提示的时候, 实际上问题就是在于你的模块里用了print...


ansible 是只检测stdout是否被污染, 所以如果我把信息写到Stderr里是没问题的. 

### python3支持问题

我们目前用的`ansible 2.6`, 我指望使用python3来进行操作, 避免一些问题. 

按照[^13]里提到的来说,  理论上会按照从哪个python版本安装的`ansible`, 就从哪个版本启动的功能. 
 

根据[^11]发现的问题就是, 我用的`2.6`通过`ansible.cfg`中的设置, 并没能生效, 而通过`-e`传入的参数倒是有效. 暂时通过上层传入来覆盖下层使用的解释器




### group_vars的读取时间是什么时候嗯
只会在执行role的时候去读取对应的`group_vars`中的变量


### 怎么将当前读入的配置及收集到的`facts`配置导出呢
* 怎么执行Export呢?
* > template - Templates a file out to a remote server
* [copy content output from json and a variable is not idempotent under py3 · Issue \#34595 · ansible/ansible](https://github.com/ansible/ansible/issues/34595#issuecomment-356091161)

### 另外, 其实我并不需要ansible原生的gather_facts, 因为他能够给出来的ip, 现在我知道一些python的原生库获取, 重点在于, 我怎么把多个节点的数据给聚合到一个节点上.
  * 通过set_fact去一起设置到一个字典里?
  * 我通过vars聚合完了.  有个map('extract'功能可以满足这套
``` yaml
- hosts: all
  vars:
    uuids: |
      {%- set o=[] %}
      {%- for i in play_hosts %}
        {%- if o.append(hostvars[i].uuid) %}
        {%- endif %}
      {%- endfor %}
      {{ o }}
  tasks:
    - name: get uuids for existing cluster nodes
      shell: mysql -N -B -u {{ db_user }} -p {{ db_user_password }} -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_state_uuid';" | sed 's/\t/,/g' | cut -f2 -d','
      register: maria_cluster_uuids
    - set_fact:
        uuid: "{{ maria_cluster_uuids.stdout }}"
    - debug:
        var: uuids
      run_once: true
      delegate_to: 127.0.0.1
```
[Is it possible combine remote results to local a register in Ansible? \- Server Fault](https://serverfault.com/questions/710010/is-it-possible-combine-remote-results-to-local-a-register-in-ansible/714455)

### vars选项的运行时间是什么时候呢? 如果我采用task, 他能在我指定task前自动运行吗?
* vars的调用时间是什么时候呢? 写个代码模拟一下?
* 目前来看, 是当用到这个变量的时候, 才会去计算
* 有没有办法把list的tuple转成dict呢?  

### jinja2模板导出yaml

并不能自动识别缩进. 需要自己来控制order, 使用`Jinja2`的`filter`可以完成这个步骤.

```
pools:
{{ pools | to_yaml | indent(2, true) }}

```

[to\_nice\_yaml output not as expected · Issue \#16707 · ansible/ansible](https://github.com/ansible/ansible/issues/16707)

# Reference
1. [ceph\-ansible 使用 \| Bolog](https://zhoubofsy.github.io/2019/09/28/storage/ceph/ceph-ansible-usage/)
2. [干货｜基于Ansible的Ceph自动化部署解析\_中兴开发者社区\-CSDN博客](https://blog.csdn.net/O4dC8OjO7ZL6/article/details/78464441)
3. [第八章: 扩展ansible\_个人文章 \- SegmentFault 思否](https://segmentfault.com/a/1190000015953427)
4. [Full support for \{\{ osd\_crush\_location \}\} · Issue \#2195 · ceph/ceph\-ansible](https://github.com/ceph/ceph-ansible/issues/2195)
5. [Easily deploy containerized Ceph daemons with Vagrant \| Sébastien Han](https://www.sebastien-han.fr/blog/2016/02/08/easily-deploy-containerized-ceph-daemons-with-vagrant/)
6. [ansible debug模块学习笔记\-行者之路\-51CTO博客](https://blog.51cto.com/hy2009/1843232)
7. [一文搞懂 ansible 变量配置 \- biglittleant \- 博客园](https://www.cnblogs.com/biglittleant/p/13072981.html)
8. [Ansible 面向企业大规模使用探究 – IBM Developer](https://developer.ibm.com/zh/depmodels/cloud/articles/cl-lo-ansible-large-scale-use-of-enterprises-explore/)
9. [可能是最强网工ansible入门及深入教程之 ansible杂谈 \- 知乎](https://zhuanlan.zhihu.com/p/147242775)
10. [Ansible最佳实践 \| Yiran's Blog](https://zdyxry.github.io/2018/11/24/Ansible%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/)
11. [ansible/python\_3\_support\.rst at stable\-2\.6 · ansible/ansible](https://github.com/ansible/ansible/blob/stable-2.6/docs/docsite/rst/reference_appendices/python_3_support.rst)
12. [Interpreter Discovery — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/interpreter_discovery.html)
13. [Wrong python2 interpreter instead of python3 · Issue \#69494 · ansible/ansible](https://github.com/ansible/ansible/issues/69494)
14. [一些小团队的自动化运维实践经验 \| 程序猿DD](https://blog.didispace.com/devops-for-small-team/)
15. [How to Manage Multistage Environments with Ansible \| DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-manage-multistage-environments-with-ansible)
16. [基于Ansible自研的可视化和并行自动化运维引擎 \- 知乎](https://zhuanlan.zhihu.com/p/291348171)
17. [Ansible 实现原理（源码分析） – Tiantian Gao \( gtt116 \)](https://www.gaott.top/dive-into-ansible-source-code/)