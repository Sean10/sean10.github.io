---
title: ceph之Nautilus版本引入的Perf统计
subtitle: nautilus_rbd_perf
date: 2020-07-06 00:15:48
updated:
tags: [ceph, perf, rbd, 源码]
categories: [专业]
---



## 背景

Nautilus在OSD和MGR中合并了通用的度量收集框架，以提供内置的监视，并且在该框架之上构建了新的RBD性能监视工具，以将各个RADOS对象度量转换为针对IOPS，吞吐量和性能的聚合RBD镜像度量。这些指标都是在Ceph集群本身内部生成和处理的，因此无需访问客户端节点即可获取指标。


## 源码阅读过程

### 新增接口
``` bash
rbd perf image iostat 
rbd perf image iotop
```


根据这里[v14\.2\.9 Nautilus — Ceph Documentation](https://docs.ceph.com/docs/master/releases/nautilus/)

>New rbd perf image iotop and rbd perf image iostat commands provide an iotop- and iostat-like IO monitor for all RBD images.

>The ceph-mgr Prometheus exporter now optionally includes an IO monitor for all RBD images.

应该是在14.2.0的发布里完成的，那这次大版本包含了什么？小版本更新会包含pr和issue链接

14版本还增加了ceph config全局配置修改的功能。

ceph14支持pg数减小和pg数根据pool容量自动调整
ceph telemetry命令用ceph mgr module enable telemetry启用，但是对telemetry不了解也跳过了。
这个应该就是我要看到的openTelemetry

[Ceph v12\.2\.13 Luminous released \- Ceph](https://ceph.io/releases/v12-2-13-luminous-released/)

这个应该是12版本最后一次补丁升级吧

这个里不包含大版本开发的代码



[rbd: implement new 'rbd perf image iostat/iotop' commands by dillaman · Pull Request \#26133 · ceph/ceph](https://github.com/ceph/ceph/pull/26133/files)

就这个提交来看，大部分代码都在处理上下文的接口适配,增加选项，假如说这套代码里只是使用已经增加的接口，那么代码里就应该有调用的API


这个问题怎么才能验证?


`src/toos/rbd/action/Perf.cc `这个新增文件，依赖的是rbd perf image stats。

然后`rbd perf image stats`也是这次commit里增加的，是继承MgrModule使用API实现的，看看这里面用的哪些api.

应该就是这里拿的了

`register_osd_perf_queries`好像有点像了。


这里使用了mgr_module.py提供的add_osd_perf_query

从这里拿到了osd层面的ops,write_ops, read_ops, bytes, write_bytes, latency这些内容

拿到osd的结果，怎么计算出rbd层的？

他这里只查appplication_metadata里有rbd这个的资源池，然后拿到资源池的osd_map

user_query变量里，QUERY_POOL_ID_MAP拿到了


在PerfHandler里进行的周期查询perf数据。

在这里周期获取数据时又调用了mgr-module的get_osd_perf_counters。emm，这个函数在12版本我们的代码里还没有，不过单论这部分呼出，其实我们可以。这个函数在14版本里，prometheus也调用了这个，那就有意思了，prometheus刚好在这个版本支持了rbd performance monitor, ceph-export输出了这部分数据。那是不是就是从这里拿到的呢？（osd_perf_query\rbd_support\prometheus

在这里ceph-mgr里面的_ceph_get_osd_perf_counters的内容。这个函数就是Cython的封装了。

get_osd_perf_counters这个封装，好像还是在ceph-mgr里提供的，果然。

在ActivePyModules.cc里又封装了一层，这里应该是从DaemonServer.cc里封装的实际逻辑了。

果然，在ActivePyModules里实例化了DaemonServer这个对象，

这里调用osd_perf_metric_collector的get_counters。OSDPerfMetricCollector这个类，应该是12版本之后给mgr好好梳理时更新出来的。

在OSDPerfMetricSubKeyDescriptor这个struct里，封装了支持的类型

卧槽，好像这里全覆盖了,在下面这里做的扩充。从Client\pool\namespace\osd\pg\object\snap都有，牛皮

[osd: support more dynamic perf query subkey types by trociny · Pull Request \#25371 · ceph/ceph](https://github.com/ceph/ceph/pull/25371/files)

那`OSDPerfMetricCollector`呢，是在这个PR里增加的？

[mgr: create shell OSD performance query class by trociny · Pull Request \#24117 · ceph/ceph](https://github.com/ceph/ceph/pull/24117)

在下面这里提到了，开启这个收集功能之后，这套功能会遇到rbd images数据过于庞大的问题，

[osd: collect client perf stats when query is enabled by trociny · Pull Request \#24265 · ceph/ceph](https://github.com/ceph/ceph/pull/24265)

再在这个类下面继续探究就有点难的感觉了。下面就越来越细，估计需要实机环境验证了。


Prior to Red Hat Storage 4, Ceph storage administrators have not had access to built-in RBD performance monitoring and metrics gathering tools.

Ceph Storage 4 now incorporates a generic metrics gathering framework within the OSDs and MGRs to provide built-in monitoring, and new RBD performance monitoring tools are built on top of this framework to translate individual RADOS object metrics into aggregated RBD image metrics for Input/Output Operations per Second (IOPS), throughput, and latency. 

好像在这篇文章里要解释到底是怎么计算的了。哦，没解释

eph ceph mgr module enable rbd_support

Noisy Neighbors and QoS看到好几次，。

到底这部分是怎么计算出来的呢？ß



[Ceph Block Performance Monitoring](https://f2.svbtle.com/ceph-block-performance-monitoring)

 the OSD-based statistical stats are the end solution since that can never provide the latencies that the client is actually experiencing.

这篇里提到这个链接，有点意思？

[Live Performance Probes \- Ceph \- Ceph](https://tracker.ceph.com/projects/ceph/wiki/Live_Performance_Probes)


 [mgr, rbd: report rbd images perf stats to mgr by Yan\-waller · Pull Request \#16071 · ceph/ceph](https://github.com/ceph/ceph/pull/16071)

 (看这篇)

目前的怀疑点是在Nautilus的版本里在osd那层新增了不少埋点数据收集

这里提到了SLA（服务等级协议service level agreement)
（提问题的人说到了想要这种客户端级别的IOPSjiankong ）

>As we know, one perfcounter metric was created in ImageCtx when we opened a rbd image , but these metrics data is scattered and reside in kinds of clients, furthermore, a rbd image could be opened simultaneously by more than one client. report these information (especially ops, bytes, latency ) to MGR may be useful.


>Ultimately, I still expect that operational indicators that we send up to mgr will mostly get reported onwards to something else (nagios, zabbix, snmp, etc), so for O(clients) monitoring jobs we should consider skipping the middleman.

这个@jcsp大佬推荐是在mgr的插件层完成这个监控的操作，然后实际上好像也的确完成了。support-rbd啊，osd-perf-count啊，都是基于ceph-mgr的插件。

>monitoring that gives them per-client throughput, latency and op type breakdown, I'm not sure how strong the push would be to implement the separate monitoring path to go get the client's view of the same set of stats.

待看[mgr/prometheus: provide RBD stats via osd dynamic perf counters by trociny · Pull Request \#25358 · ceph/ceph](https://github.com/ceph/ceph/pull/25358)

在module.py里通过OSD_PERF_QUERY_COUNTERS_INDICES这个来把一个counter按照我们要的key导出数据。

`extract_pool_key`是用来提取像`pool1/image0`这样的spec字符串里的pool name的，

`user_queries`到底是放了什么内容？


`OSDPerfMetricCollectorListener`这个应该是才是真实被Ceph-mgr实例化的内容。、

``` c++
 class OSDPerfMetricCollectorListener :
      public OSDPerfMetricCollector::Listener

```

在`DaemonServer`初始化的时候

``` c++
osd_perf_metric_collector_listener(this),
      osd_perf_metric_collector(osd_perf_metric_collector_listener)

typedef int OSDPerfMetricQueryID;

typedef std::pair<uint64_t,uint64_t> PerformanceCounter;
typedef std::vector<PerformanceCounter> PerformanceCounters;


struct PerformanceCounterDescriptor {
      case PerformanceCounterType::OPS:
    case PerformanceCounterType::WRITE_OPS:
    case PerformanceCounterType::READ_OPS:
    case PerformanceCounterType::BYTES:
    case PerformanceCounterType::WRITE_BYTES:
    case PerformanceCounterType::READ_BYTES:
    case PerformanceCounterType::LATENCY:
    case PerformanceCounterType::WRITE_LATENCY:
    case PerformanceCounterType::READ_LATENCY:


typedef std::vector<std::string> OSDPerfMetricSubKey; // array of regex match
typedef std::vector<OSDPerfMetricSubKey> OSDPerfMetricKey;

enum class OSDPerfMetricSubKeyType : uint8_t {
  CLIENT_ID = 0,
  CLIENT_ADDRESS = 1,
  POOL_ID = 2,
  NAMESPACE = 3,
  OSD_ID = 4,
  PG_ID = 5,
  OBJECT_NAME = 6,
  SNAP_ID = 7,
};
```

根据这里来看，果然是在Nautuil版本里，给ceph-mgr里增加了很多功能。
``` c++
  typedef std::map<OSDPerfMetricQueryID,
                   std::map<OSDPerfMetricKey, PerformanceCounters>> Counters;

int OSDPerfMetricCollector::get_counters(
    OSDPerfMetricQueryID query_id,
    std::map<OSDPerfMetricKey, PerformanceCounters> *c) {


int DaemonServer::get_osd_perf_counters(
```
这里调用到`get_counter`.这个接口被封装进pybind里了。

这里面的数据都是从哪里手机的呢？这个collector只是作为一个接口层提供收集的函数，手机的内容是通过指针穿进去的。还是存在了DaemonServer这个类里。

这里面到底调用了什么接口来收集osd\rados那些的数据呢？
`OSDPerfMetricQueryID OSDPerfMetricCollector::add_query(`
在这个函数里添加的到底是任务，还是收集的值呢？应该是任务吧？

``` c++
typedef std::map<OSDPerfMetricQuery,
                   std::map<OSDPerfMetricQueryID, OptionalLimit>> Queries;
```
这个和queries有什么关系呢？
```c++
      typedef std::optional<OSDPerfMetricLimit> OptionalLimit;
  typedef std::map<OSDPerfMetricQuery,
                   std::map<OSDPerfMetricQueryID, OptionalLimit>> Queries;
  typedef std::map<OSDPerfMetricQueryID,
                   std::map<OSDPerfMetricKey, PerformanceCounters>> Counters;

  
struct OSDPerfMetricLimit {
  PerformanceCounterDescriptor order_by;
  uint64_t max_count = 0;
``` 



在`bool DaemonServer::handle_report(MMgrReport *m)`这个里
`    osd_perf_metric_collector.process_reports(m->osd_perf_metric_reports);
`
``` c++
  class OSDPerfMetricCollectorListener :
      public OSDPerfMetricCollector::Listener {
  public:
    OSDPerfMetricCollectorListener(DaemonServer *server)
      : server(server) {
    }
    void handle_query_updated() override {
      server->handle_osd_perf_metric_query_updated();
    }
```
这里继承了Listener里的虚函数接口

``` c++
void DaemonServer::handle_osd_perf_metric_query_updated()
{
  dout(10) << dendl;

  // Send a fresh MMgrConfigure to all clients, so that they can follow
  // the new policy for transmitting stats
  finisher.queue(new FunctionContext([this](int r) {
        std::lock_guard l(lock);
        for (auto &c : daemon_connections) {
          if (c->peer_is_osd()) {
            _send_configure(c);
          }
        }
      }));
}



/**
 * This message is sent from ceph-mgr to MgrClient, instructing it
 * it about what data to send back to ceph-mgr at what frequency.
 */
class MMgrConfigure : public MessageInstance<MMgrConfigure> {


```
[mgr: update MMgrConfigure message to include optional OSD perf queries by colletj · Pull Request \#24180 · ceph/ceph](https://github.com/ceph/ceph/pull/24180)

可能数据是从MgrClient里来的？嗯，忘了之前看过, 现在也才意识到mgr其实分成了Server和client，

osd和pg都看到了，但是其他的呢？如果只有这两个数据，那其他的rados那些是怎么算出来的？

set_perf_queries_cb(m->osd_perf_metric_queries);



[Ceph Manager Overview](http://blog.wjin.org/posts/ceph-manager-overview.html)

这篇里主要讲的mgr的代码

>以osd daemon为例，在启动过程中，会发送消息MMgrOpen，mgr收到后，会回复MMgrConfigure消息，主要是返回一个period时间，后续osd就根据设定的period， 定期将状态信息上报给mgr，即消息MMgrReport和MPGStats:

本以为这段代码里不涉及image的信息了

但是下面这段查询到的数据里显然带上了rbd的image信息
``` py
            res = self.module.get_osd_perf_counters(query_id)
            for counter in res['counters']:
                    raw_image[0] = None
                if not raw_image[0]:
                    raw_image[0] = [now_ts, [int(x[0]) for x in counter['c']]]

```

[mgr: templatize metrics collection interface by vshankar · Pull Request \#29214 · ceph/ceph](https://github.com/ceph/ceph/pull/29214)

这里做了一层封装模板化

blame

Daemon里的这个函数get_osd_perf_counters是下面这条里加的

[mgr: make dynamic osd perf counters accessible from modules · vshankar/ceph@b8362d9](https://github.com/vshankar/ceph/commit/b8362d904a710c10ab72f15c78fdafc3ff21f020)


get_counters这个实际工作的方法呢？

[mgr: store osd perf counters received in osd reports · vshankar/ceph@438a3f7](https://github.com/vshankar/ceph/commit/438a3f7bc40c66b14b7f61959f8dd691ea0b5220)

又没头绪了

还是回到prometheus这里使用上看看阿布

[mgr/prometheus: provide RBD stats via osd dynamic perf counters by trociny · Pull Request \#25358 · ceph/ceph](https://github.com/ceph/ceph/pull/25358/files)
``` py
  # Per RBD image stats is collected by registering a dynamic osd perf
        # stats query that tells OSDs to group stats for requests associated
        # with RBD objects by pool, namespace, and image id, which are
        # extracted from the request object names or other attributes.
        # The RBD object names have the following prefixes:
        #   - rbd_data.{image_id}. (data stored in the same pool as metadata)
        #   - rbd_data.{pool_id}.{image_id}. (data stored in a dedicated data pool)
        #   - journal_data.{pool_id}.{image_id}. (journal if journaling is enabled)
        # The pool_id in the object name is the id of the pool with the image
        # metdata, and should be used in the image spec. If there is no pool_id
        # in the object name, the image pool is the pool where the object is
        # located.

              if 'query_id' not in self.rbd_stats:
            query = {
                'key_descriptor': [
                    {'type': 'pool_id', 'regex': pool_id_regex},
                    {'type': 'namespace', 'regex': namespace_regex},
                    {'type': 'object_name',
                     'regex': '^(?:rbd|journal)_data\.(?:([0-9]+)\.)?([^.]+)\.'},
                ],
                'performance_counter_descriptors': list(counters_info),
            }
            query_id = self.add_osd_perf_query(query)
            if query_id is None:
                self.log.error('failed to add query %s' % query)
                return
            self.rbd_stats['query'] = query
            self.rbd_stats['query_id'] = query_id

        res = self.get_osd_perf_counters(self.rbd_stats['query_id'])
```
这段注释写的是真的好

所以其实还是这个借口的功能，类似mon_command，提供了根据传入的数据进行收集的操作。

这里也的确用上赋值逻辑

  queryID到底是怎么期作用的？

  来自`add_osd_perf_query`

OSDPerfMetricQueryID DaemonServer::add_osd_perf_query(

``` c++
              query = {
                'key_descriptor': [
                    {'type': 'pool_id', 'regex': pool_id_regex},
                    {'type': 'namespace', 'regex': namespace_regex},
                    {'type': 'object_name',
                     'regex': '^(?:rbd|journal)_data\.(?:([0-9]+)\.)?([^.]+)\.'},
                ],
                'performance_counter_descriptors': list(counters_info),
            }
            query_id = self.add_osd_perf_query(query)
``` 

使用上

[mgr: create shell OSD performance query class · vshankar/ceph@a6c3390](https://github.com/vshankar/ceph/commit/a6c3390834759e3f953962d46cc7c840c9970046)

这个`add_osd_perf_query`是在这里加的

add_query也是他家的。
``` c++
static PyObject*
ceph_add_osd_perf_query(BaseMgrModule *self, PyObject *args)
{
  static const std::string NAME_KEY_DESCRIPTOR = "key_descriptor";
  static const std::string NAME_COUNTERS_DESCRIPTORS =
      "performance_counter_descriptors";
  static const std::string NAME_LIMIT = "limit";
  static const std::string NAME_SUB_KEY_TYPE = "type";
  static const std::string NAME_SUB_KEY_REGEX = "regex";
  static const std::string NAME_LIMIT_ORDER_BY = "order_by";
  static const std::string NAME_LIMIT_MAX_COUNT = "max_count";
``` 
  在封装python这里做的参数解析
``` C++
auto query_id = self->py_modules->add_osd_perf_query(query, limit);
```
所有属性都传到了limit里

在_send_configure的时候，把limit里的任务传给了MgrClient
``` c++
struct OSDPerfMetricLimit {
  PerformanceCounterDescriptor order_by;
  uint64_t max_count = 0;

  OSDPerfMetricLimit() {
  }

  OSDPerfMetricLimit(const PerformanceCounterDescriptor &order_by,
                     uint64_t max_count)
    : order_by(order_by), max_count(max_count) {
  }

  bool operator<(const OSDPerfMetricLimit &other) const {
    if (order_by != other.order_by) {
      return order_by < other.order_by;
    }
    return max_count < other.max_count;
  }

  DENC(OSDPerfMetricLimit, v, p) {
    DENC_START(1, 1, p);
    denc(v.order_by, p);
    denc(v.max_count, p);
    DENC_FINISH(p);
  }
};
```
传过去的应该是这个`OptionalLimit`吧，这个有啥用？

里面有个`order_by`里申明是哪种type,是pg还是rados了好像


encode之后发送到mgrclient

看看怎么解码

  std::function<void(const std::map<OSDPerfMetricQuery,
                                    OSDPerfMetricLimits> &)> set_perf_queries_cb;

在这个`MgrClient.cc`里居然有2个`set_perf_queries_cb`

一个是封装的MgrClient对外暴露的函数，就是用来赋值的？

一个是private的值，与mgr的Daemon沟通时进行调用。

但是这里好像只看到了osd.cc触发啊

这个被osd.cc里调用了


```c++
void OSD::set_perf_queries(
    const std::map<OSDPerfMetricQuery, OSDPerfMetricLimits> &queries) {
```

怎么看上去osd.cc里不止负责普通的osd信息收集

在12版本里，这个OSD.cc里也有mgrc

在14版本里，  std::list<OSDPerfMetricQuery> m_perf_queries;属于OSD这个class
``` c++
private:
  void set_perf_queries(
      const std::map<OSDPerfMetricQuery, OSDPerfMetricLimits> &queries);
  void get_perf_reports(
      std::map<OSDPerfMetricQuery, OSDPerfMetricReport> *reports);

  Mutex m_perf_queries_lock = {"OSD::m_perf_queries_lock"};
  std::list<OSDPerfMetricQuery> m_perf_queries;
  std::map<OSDPerfMetricQuery, OSDPerfMetricLimits> m_perf_limits;

  void get_perf_reports(
      std::map<OSDPerfMetricQuery, OSDPerfMetricReport> *reports);



  mgrc.set_pgstats_cb([this](){ return collect_pg_stats(); });
  mgrc.set_perf_metric_query_cb(
    [this](const std::map<OSDPerfMetricQuery, OSDPerfMetricLimits> &queries) {
        set_perf_queries(queries);
      },
      [this](std::map<OSDPerfMetricQuery, OSDPerfMetricReport> *reports) {
        get_perf_reports(reports);
      });
  mgrc.init();
```

osd把收集pg信息的回调函数设置到mgrc里,接下来应该是由mgrc来调用了把

在`void MgrClient::_send_report()`这里会触发调用.

理论上应该是

``` c++

void OSD::get_perf_reports(
    std::map<OSDPerfMetricQuery, OSDPerfMetricReport> *reports) {
  std::vector<PGRef> pgs;
  _get_pgs(&pgs);
  DynamicPerfStats dps;
  for (auto& pg : pgs) {
    // m_perf_queries can be modified only in set_perf_queries by mgr client
    // request, and it is protected by by mgr client's lock, which is held
    // when set_perf_queries/get_perf_reports are called, so we may not hold
    // m_perf_queries_lock here.
    DynamicPerfStats pg_dps(m_perf_queries);
    pg->lock();
    pg->get_dynamic_perf_stats(&pg_dps);
    pg->unlock();
    dps.merge(pg_dps);
  }
  dps.add_to_reports(m_perf_limits, reports);
  dout(20) << "reports for " << reports->size() << " queries" << dendl;
}
```
  void add_to_reports(

下面这个数据在primary_log的里
``` c++
class PrimaryLogPG : public PG, public PGBackend::Listener {

  DynamicPerfStats m_dynamic_perf_stats;


void PrimaryLogPG::get_dynamic_perf_stats(DynamicPerfStats *stats)

  std::swap(m_dynamic_perf_stats, *stats);
```
而这个value在下面这里更新的
```c++
void PrimaryLogPG::log_op_stats(const OpRequest& op,

    m_dynamic_perf_stats.add(osd, info, op, inb, outb, latency);
```
差别可能真在这?在12版本的这个`log_op_stats`函数里,没有记录perf信息的.
``` c++
          auto m = static_cast<const MOSDOp*>(op.get_req());
std::string match_string;
          switch(d.type) {
          case OSDPerfMetricSubKeyType::CLIENT_ID:
            match_string = stringify(m->get_reqid().name);
            break;
          case OSDPerfMetricSubKeyType::CLIENT_ADDRESS:
            match_string = stringify(m->get_connection()->get_peer_addr());
            break;
          case OSDPerfMetricSubKeyType::POOL_ID:
            match_string = stringify(m->get_spg().pool());
            break;
          case OSDPerfMetricSubKeyType::NAMESPACE:
            match_string = m->get_hobj().nspace;
            break;
          case OSDPerfMetricSubKeyType::OSD_ID:
            match_string = stringify(osd->get_nodeid());
            break;
          case OSDPerfMetricSubKeyType::PG_ID:
            match_string = stringify(pg_info.pgid);
            break;
          case OSDPerfMetricSubKeyType::OBJECT_NAME:
            match_string = m->get_oid().name;
            break;
          case OSDPerfMetricSubKeyType::SNAP_ID:
            match_string = stringify(m->get_snapid());
            break;
          default:
            ceph_abort_msg("unknown counter type");
          }
```
的确是在这个add这里加的,奇怪了.这里的`m`是怎么拿到所谓的snap\rados object这些信息的呢?
``` c++
          auto m = static_cast<const MOSDOp*>(op.get_req());

Requests are MOSDOp messages.  Replies are MOSDOpReply messages.

An object request is targeted at an hobject_t, which includes a pool,
hash value, object name, placement key (usually empty), and snapid.

The hash value is a 32-bit hash value, normally generated by hashing
the object name.  The hobject_t can be arbitrarily constructed,
though, with any hash value and name.  Note that in the MOSDOp these
components are spread across several fields and not logically
assembled in an actual hobject_t member (mainly historical reasons).

A request can also target a PG.  In this case, the *ps* value matches
a specific PG, the object name is empty, and (hopefully) the ops in
the request are PG ops.

Either way, the request ultimately targets a PG, either by using the
explicit pgid or by folding the hash value onto the current number of
pgs in the pool.  The client sends the request to the primary for the
associated PG.

Each request is assigned a unique tid.
```
原来这里是osd的落盘请求链路,这里能通过req()
``` c++
class MOSDOp : public MessageInstance<MOSDOp, MOSDFastDispatchOp> {


  std::map<OSDPerfMetricQuery,
           std::map<OSDPerfMetricKey, PerformanceCounters>> data;
```
  [osd: collect client perf stats when query is enabled by trociny · Pull Request \#24265 · ceph/ceph](https://github.com/ceph/ceph/pull/24265/files#)

找到了,是在这个pr里增加的数据收集, 又是`trociny`这个大佬提交的.

  ceph tell mgr osd perf query add simple the mgr starts to receive perf report, writing to the log:

  简单点说原理是在osd的最下层通过request的MOSDop这个类,查到这次request各层的名字,然后在对应的计数器数加一
``` c++
  void PrimaryLogPG::log_op_stats(const OpRequest& op,
				const uint64_t inb,
				const uint64_t outb)
```
在这的时候传入的op.

这个DynamicPerfStats.h的文件是在osd目录里.

    dps.merge(pg_dps);




