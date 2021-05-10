---
title: ceph之mon_client时间长排查
subtitle: tech
date: 2021-04-26 19:56:42
updated:
tags: [ceph, mon, 超时]
categories: [专业]
---

# 背景
在使用`12.2.1`版本时, 发现经常会出现需要10+秒才能返回`ceph -s`的现象, 偶尔又`1s`内就返回了.


# 排查

首先, `time ceph -s --debug_monc 40 --debug_ms 40`看下详细日志有没有记录在哪呢?

看了下, 应该是在跟mon通信过程, 但是好像不知道发生了什么.

咨询了一下大神给了个关键词, `objecter`, 打开`debug`后看到一部分内容了. 有resend_mon_ops的操作, 那就代表第一次`picked`到的`ip`, 有些没正常工作?
## 异常日志
```
2021-03-08 17:46:40.591805 7ff668ff9700  1 monclient(hunting): continuing hunt                                                                                                               [47/3291]2021-03-08 17:46:40.591809 7ff668ff9700 10 monclient(hunting): _reopen_session rank -1
2021-03-08 17:46:40.592032 7ff668ff9700 10 monclient(hunting): picked mon.noname-b con 0x7ff640001210 addr 10.192.89.63:6789/0
2021-03-08 17:46:40.592112 7ff668ff9700 10 monclient(hunting): picked mon.noname-c con 0x7ff640005ad0 addr 10.192.89.61:6789/0
2021-03-08 17:46:40.592194 7ff668ff9700 10 monclient(hunting): _renew_subs
2021-03-08 17:46:40.593008 7ff669ffb700 10 client.?.objecter ms_handle_connect 0x7ff640005ad0
2021-03-08 17:46:40.593072 7ff669ffb700 10 client.?.objecter resend_mon_ops
2021-03-08 17:46:40.593083 7ff669ffb700 10 client.?.objecter ms_handle_connect 0x7ff640001210
2021-03-08 17:46:40.593104 7ff669ffb700 10 client.?.objecter resend_mon_ops
2021-03-08 17:46:42.592257 7ff668ff9700 10 monclient(hunting): tick
2021-03-08 17:46:42.592283 7ff668ff9700  1 monclient(hunting): continuing hunt
2021-03-08 17:46:42.592287 7ff668ff9700 10 monclient(hunting): _reopen_session rank -1
2021-03-08 17:46:42.592435 7ff668ff9700 10 monclient(hunting): picked mon.noname-a con 0x7ff64000d0b0 addr 10.192.89.60:6789/0
2021-03-08 17:46:42.592468 7ff668ff9700 10 monclient(hunting): picked mon.noname-c con 0x7ff6400106f0 addr 10.192.89.61:6789/0
2021-03-08 17:46:42.592536 7ff668ff9700 10 monclient(hunting): _renew_subs
2021-03-08 17:46:42.593106 7ff669ffb700 10 client.?.objecter ms_handle_connect 0x7ff64000d0b0
2021-03-08 17:46:42.593133 7ff669ffb700 10 client.?.objecter resend_mon_ops
2021-03-08 17:46:42.593140 7ff669ffb700 10 client.?.objecter ms_handle_connect 0x7ff6400106f0
2021-03-08 17:46:42.593142 7ff669ffb700 10 client.?.objecter resend_mon_ops
2021-03-08 17:46:42.593596 7ff669ffb700 10 monclient(hunting): handle_monmap mon_map magic: 0 v1
2021-03-08 17:46:42.593638 7ff669ffb700 10 monclient(hunting):  got monmap 1, mon.noname-a is now rank -1
2021-03-08 17:46:42.593643 7ff669ffb700 10 monclient(hunting): dump:
```
### 正常的op日志
```
d = 0
2021-03-08 18:50:37.595153 7fe83ffff700 30 Event(0x7fe8400e8650 nevent=5000 time_id=2).create_time_event id=1 trigger after 900000000us
2021-03-08 18:50:37.595151 7fe83f7fe700 20 -- 10.192.89.60:0/288270703 >> 10.192.89.60:6789/0 conn(0x7fe8403a2660 :-1 s=STATE_OPEN_MESSAGE_READ_FOOTER_AND_DISPATCH pgs=50114 cs=1 l=1).process got 397 + 0 + 0 byte message
2021-03-08 18:50:37.595158 7fe83ffff700 30 Event(0x7fe8400e8650 nevent=5000 time_id=2).dispatch_event_external 0x7fe84039c280 pending 1
2021-03-08 18:50:37.595160 7fe80affd700 10 client.?.objecter ms_handle_connect 0x7fe84039ef30
2021-03-08 18:50:37.595162 7fe80affd700 10 client.?.objecter resend_mon_ops
2021-03-08 18:50:37.595174 7fe83ffff700 20 -- 10.192.89.60:0/288270703 >> 10.192.89.63:6789/0 conn(0x7fe84039ef30 :-1 s=STATE_OPEN pgs=27784 cs=1 l=1).process prev state is STATE_CONNECTING_READY
2021-03-08 18:50:37.595181 7fe83ffff700 25 -- 10.192.89.60:0/288270703 >> 10.192.89.63:6789/0 conn(0x7fe84039ef30 :-1 s=STATE_OPEN pgs=27784 cs=1 l=1).read_until len is 1 state_offset is 0
2021-03-08 18:50:37.595184 7fe83f7fe700 10 -- 10.192.89.60:0/288270703 >> 10.192.89.60:6789/0 conn(0x7fe8403a2660 :-1 s=STATE_OPEN_MESSAGE_READ_FOOTER_AND_DISPATCH pgs=50114 cs=1 l=1).process no session security set
2021-03-08 18:50:37.595190 7fe83ffff700 25 -- 10.192.89.60:0/288270703 >> 10.192.89.63:6789/0 conn(0x7fe84039ef30 :-1 s=STATE_OPEN pgs=27784 cs=1 l=1).read_until read_bulk recv_end is 0 left is 1 got 0
2021-03-08 18:50:37.595196 7fe83ffff700 25 -- 10.192.89.60:0/288270703 >> 10.192.89.63:6789/0 conn(0x7fe84039ef30 :-1 s=STATE_OPEN pgs=27784 cs=1 l=1).read_until need len 1 remaining 1 bytes
2021-03-08 18:50:37.595193 7fe83f7fe700  5 -- 10.192.89.60:0/288270703 >> 10.192.89.60:6789/0 conn(0x7fe8403a2660 :-1 s=STATE_OPEN_MESSAGE_READ_FOOTER_AND_DISPATCH pgs=50114 cs=1 l=1). rx mon.0 seq
1 0x7fe83000b830 mon_map magic: 0 v1
2021-03-08 18:50:37.595203 7fe83ffff700 10 -- 10.192.89.60:0/288270703 >> 10.192.89.63:6789/0 conn(0x7fe84039ef30 :-1 s=STATE_OPEN pgs=27784 cs=1 l=1).handle_write
```

刚才忽然想到的, 会不会是因为有些时候先去访问了非主节点, 所以没响应, 直到去访问主节点为止.

mon为什么非主节点就不能提供信息呢?理论上应该是全节点都可以(PS. 后来试了下`12.2.12`版本, 全节点都正常, 就这个版本不正常, 暂时没去查这个版本到`12.2.12`改了什么)

所以monclient是怎么选择`mon_host`的呢?

mon_client是怎么触发的呢?


* m_client->dump_status(f);
* bool Client::CommandHook::call 
  * 这个是怎么进来的呢?
  * rados什么时候会进入这个client::init?
* static CephContext *rados_create_cct(const char * const clustername,
                                     CephInitParameters *iparams)
* CephContext::CephContext(uint32_t module_type_,
                         enum code_environment_t code_env,
                         int init_flags_)
*  _admin_hook = new CephContextHook(this);
* *pcluster = reinterpret_cast<rados_t>(new librados::RadosClient(cct));
* librados::RadosClient::RadosClient(CephContext *cct_)
* MonClient::MonClient(CephContext *cct_)

现在问题就是init之外, ceph -s怎么调用的status了.

按理来说, 只是把`status`发送给了mon, 来进行mon_command

但是现在mon_host, 他其实是不知道的. emm, 按这个来说, 应该再init阶段, 或者connect阶段知道的

* rados_connect
* int MonMap::build_initial(CephContext *cct, ostream& errout)

的确是在connect阶段, 从build_initial中取得的.
``` c++
int MonMap::build_from_host_list(std::string hostlist, std::string prefix)

  void add(const string &name, const entity_addr_t &addr) {
    add(mon_info_t(name, addr));

    struct mon_info_t {
```


接下来就逐渐进到这个pick阶段了

``` c++
        ldout(cct, 10) << "picked mon." << monmap.get_name(rank)
```

开始反查

``` c++
void MonClient::_add_conns(uint64_t global_id)

    std::random_device rd;
    std::mt19937 rng(rd());

    std::shuffle(ranks.begin(), ranks.end(), rng);

void MonClient::_reopen_session(int rank)
``` 

这里的n如果一开始就是集群size, 好像即便shuffle也不会有问题了.

 cat /etc/ceph/ceph.conf | grep  mon_client_hunt_parallel

     unsigned n = cct->_conf->mon_client_hunt_parallel;

的确找到了对应的,我把这个size改成`mon_host`的`size`, 一开始就一定能选中主节点, 就不卡了.


