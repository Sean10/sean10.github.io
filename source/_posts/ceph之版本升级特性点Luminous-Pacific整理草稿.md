---
title: ceph之版本升级特性点Luminous_Pacific整理草稿
subtitle: tech
date: 2023-06-09 23:19:04
updated:
tags: [ceph, upgrade]
categories: [专业]
---


# 背景
主要是针对rbd->openstack相关, 所以cephfs/rgw/dashboard等其他的特性相关的几乎都跳过了. 仅整理了rbd及以下模块, 从luminous以后到pacific为止的.

# 性能相关

## Pacific

### 16.2.0

RADOS

* Pacific introduces :ref:`bluestore-rocksdb-sharding`, which reduces disk space requirements.
* Ceph now provides QoS between client I/O and background operations via the
  mclock scheduler.

RBD

* A new persistent write-back cache is available.  The cache operates in
  a log-structured manner, providing full point-in-time consistency for the
  backing image.  It should be particularly suitable for PMEM devices.
[pacific: librbd/cache/pwl: persistant cache backports by ideepika · Pull Request #43772 · ceph/ceph](https://github.com/ceph/ceph/pull/43772)


## Octopus

### 15.2.15

* The default value of `osd_client_message_cap` has been set to 256, to provide
  better flow control by limiting maximum number of in-flight client requests.

https://hub.nuaa.cf/ceph/ceph/pull/42616

https://docs.google.com/spreadsheets/d/1dwKcxFKpAOWzDPekgojrJhfiCtPgiIf8CGGMG1rboRU/edit#gid=0


这里的实测结论居然是256比较好? 还没细看.

### 15.2.0

RADOS

* Objects can now be brought in sync during recovery by copying only
  the modified portion of the object, reducing tail latencies during
  recovery.
	  [osd: partial recovery strategy based on PGLog by mslovy · Pull Request #21722 · ceph/ceph](https://github.com/ceph/ceph/pull/21722)
* BlueStore has received several improvements and performance
  updates, including improved accounting for "omap" (key/value)
  object data by pool, improved cache memory management, and a
  reduced allocation unit size for SSD devices.  (Note that by
  default, the first time each OSD starts after upgrading to octopus
  it will trigger a conversion that may take from a few minutes to a
  few hours, depending on the amount of stored "omap" data.)
* Snapshot trimming metadata is now managed in a more efficient and
  scalable fashion.

RBD

* Clone operations now preserve the sparseness of the underlying RBD image.
	* [librbd: clone copy-on-write operations should preserve sparseness by trociny · Pull Request #27999 · ceph/ceph](https://github.com/ceph/ceph/pull/27999)





## Nautilus

### 14.2.22

* This release sets ``bluefs_buffered_io`` to true by default to improve performance
  for metadata heavy workloads. Enabling this option has been reported to
  occasionally cause excessive kernel swapping under certain workloads.
  Currently, the most consistent performing combination is to enable
  bluefs_buffered_io and disable system level swap.

* The default value of ``bluestore_cache_trim_max_skip_pinned`` has been
  increased to 1000 to control memory growth due to onodes.

### 14.2.8

* The default value of ``bluestore_min_alloc_size_ssd`` has been changed to 4K to improve performance across all workloads.
> 	https://github.com/ceph/ceph/pull/32998
> [common/options: Set bluestore min_alloc size to 4K by markhpc · Pull Request #30698 · ceph/ceph](https://github.com/ceph/ceph/pull/30698)
> 
> 
> I'm not sure that having 4K min_alloc_size for both ssd and spinner is a good choice. From my benchmarks 4K MAS is undoubtedly beneficial in all-flash setupы only. It shows performance boost for certain small R/W scenarios and no regression for other ones  
> When main device is rotational performance boost is observed for a minor selection of 4K R/W scenarios. But other scenarios suffer from pretty large performance degradation.
> 
> See columns B & C, rows 20-36 at  
> [https://docs.google.com/spreadsheets/d/1xXbheoGwgyaLBc389TFo1vUyz9LjshI83_bW8zBYWl8/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1xXbheoGwgyaLBc389TFo1vUyz9LjshI83_bW8zBYWl8/edit?usp=sharing)  
> So my suggestion is to tune bluestore_min_alloc_size_ssd only
> 

### 14.2.0 


  * The NUMA node for OSD daemons can easily be monitored via the
    ``ceph osd numa-status`` command, and configured via the
    ``osd_numa_node`` config option.
[Bug #42411: nautilus:osd: network numa affinity not supporting subnet port - RADOS - Ceph](https://tracker.ceph.com/issues/42411)

## Mimic



### 13.2.0

- *RADOS*:

  * An *async recovery* feature reduces the tail latency of requests
    when the OSDs are recovering from a recent failure.
	    https://github.com/ceph/ceph/pull/19811
  * OSD preemption of scrub by conflicting requests reduces tail latency.   (12版本已有, 2017年的修改)
	  * [osd/PG: allow scrub preemption by liewegas · Pull Request #18971 · ceph/ceph](https://github.com/ceph/ceph/pull/18971)


# 其他功能性

### 16.2.11

- Trimming of PGLog dups is now controlled by the size instead of the version. This fixes the PGLog inflation issue that was happening when the on-line (in OSD) trimming got jammed after a PG split operation. Also, a new off-line mechanism has been added: ceph-objectstore-tool got trim-pg-log-dups op that targets situations where OSD is unable to boot due to those inflated dups. If that is the case, in OSD logs the “You can be hit by THE DUPS BUG” warning will be visible. Relevant tracker: [https://tracker.ceph.com/issues/53729](https://tracker.ceph.com/issues/53729)
    
- RBD: rbd device unmap command gained --namespace option. Support for namespaces was added to RBD in Nautilus 14.2.0 and it has been possible to map and unmap images in namespaces using the image-spec syntax since then but the corresponding option available in most other commands was missing.
	rados namespaces can be used to provide isolation between rados clients within a pool. For example, a client could only have full permissions on a namespace specific to them. This makes using a different rados client for each tenant feasible, which is particularly useful for rbd where many different tenants are accessing their own rbd images.
		即主要用途其实算是更精细的权限控制? 确保底层即便拿到其他用户的权限, 也因为namespace无法使用?


### 16.2.2
- Cephadm now supports an _ingress_ service type that provides load balancing and HA (via haproxy and keepalived on a virtual IP) for RGW service (see [High availability service for RGW](https://docs.ceph.com/en/latest/cephadm/services/rgw/#orchestrator-haproxy-service-spec)). (The experimental _rgw-ha_ service has been removed.)

### 16.2.0

- Official [Ceph RESTful API](https://docs.ceph.com/en/latest/mgr/ceph_api/#mgr-ceph-api):
    - OpenAPI v3 compliant.        
    - Stability commitment starting from Pacific release.
    - Versioned via HTTP `Accept` header (starting with v1.0).
    - Thoroughly tested (>90% coverage and per Pull Request validation).
    - Fully documented.

* Security (multiple enhancements and fixes resulting from a pen testing conducted by IBM):

Account lock-out after a configurable number of failed log-in attempts.

Improved cookie policies to mitigate XSS/CSRF attacks.

Reviewed and improved security in HTTP headers.

Sensitive information reviewed and removed from logs and error messages.

TLS 1.0 and 1.1 support disabled.

Debug mode when enabled triggers HEALTH_WARN.

* CLAY CODE PLUGIN
	* ec池, 故障修复时, 可以减少对带宽的占用.
	* [CLAY code plugin — Ceph Documentation](https://docs.ceph.com/en/latest/rados/operations/erasure-code-clay/#:~:text=CLAY%20code%20plugin%20CLAY%20%28short%20for%20coupled-layer%29%20codes,IO%20when%20a%20failed%20node%2FOSD%2Frack%20is%20being%20repaired.)

SPECIFYING EXPECTED POOL SIZE
	支持多池共用osd, 做健康检查, 感知是否使用超过该池原定使用空间上限, 会有warning信息.
ceph osd pool set {pool-name} recovery_priority {value}
rbd 性能监控
- Image live-migration feature has been extended to support external data sources. Images can now be instantly imported from local files, remote files served over HTTP(S) or remote S3 buckets in `raw` (`rbd export v1`) or basic `qcow` and `qcow2` formats. Support for `rbd export v2` format, advanced QCOW features and `rbd export-diff` snapshot differentials is expected in future releases.
    
- Initial support for client-side encryption has been added. This is based on LUKS and in future releases will allow using per-image encryption keys while maintaining snapshot and clone functionality -- so that parent image and potentially multiple clone images can be encrypted with different keys.
- - A Windows client is now available in the form of `librbd.dll` and `rbd-wnbd` (Windows Network Block Device) daemon. It allows mapping, unmapping and manipulating images similar to `rbd-nbd`.
- - librbd API now offers quiesce/unquiesce hooks, allowing for coordinated snapshot creation.
- A new library is available, libcephsqlite. It provides a SQLite Virtual File System (VFS) on top of RADOS. The database and journals are striped over RADOS across multiple objects for virtually unlimited scaling and throughput only limited by the SQLite client. Applications using SQLite may change to the Ceph VFS with minimal changes, usually just by specifying the alternate VFS. We expect the library to be most impactful and useful for applications that were storing state in RADOS omap, especially without striping which limits scalability.
	- 提供了基本的数据库
- 重构有了预计时间?
- 提供了静默rbd的hook接口
	- rbd device --device-type nbd map --quiesce --quiesce-hook ${QUIESCE_HOOK} \
	- 看了下16版本有提供rbd quiesce 逻辑, 针对snap create时可以主动block rbd块的io, 并提供静默io前后的hook. 没找着有写测试/使用样例, 不过rbd-nbd那边有写用例, 可以提供脚本做前置/后置检查.


### 15.2.15

* A new ceph-erasure-code-tool has been added to help manually recover an
  object from a damaged PG.


### 14.2.22

* A ``--max <n>`` option is available with the ``osd ok-to-stop`` command to
  provide up to N OSDs that can be stopped together without making PGs
  unavailable.

msgr2下的arm/x86不兼容修复
* A long-standing bug that prevented 32-bit and 64-bit client/server
  interoperability under msgr v2 has been fixed.  In particular, mixing armv7l
  (armhf) and x86_64 or aarch64 servers in the same cluster now works.

### 14.2.5


OSD:

* A new OSD daemon command, 'dump_recovery_reservations', reveals the
  recovery locks held (in_progress) and waiting in priority queues.


* A health warning is now generated if the average osd heartbeat ping
  time exceeds a configurable threshold for any of the intervals
  computed. The OSD computes 1 minute, 5 minute and 15 minute
  intervals with average, minimum and maximum values.  New configuration
  option `mon_warn_on_slow_ping_ratio` specifies a percentage of
  `osd_heartbeat_grace` to determine the threshold.  A value of zero
  disables the warning. New configuration option `mon_warn_on_slow_ping_time`
  specified in milliseconds over-rides the computed value, causes a warning
  when OSD heartbeat pings take longer than the specified amount.
  A new admin command, `ceph daemon mgr.# dump_osd_network [threshold]`, will
  list all connections with a ping time longer than the specified threshold or
  value determined by the config options, for the average for any of the 3 intervals.
  Another new admin command, `ceph daemon osd.# dump_osd_network [threshold]`,
  will do the same but only including heartbeats initiated by the specified OSD



* Ceph will now issue health warnings if daemons have recently crashed. Ceph
  has been collecting crash reports since the initial Nautilus release, but the
  health alerts are new. To view new crashes (or all crashes, if you've just
  upgraded)::

    ceph crash ls-new

  To acknowledge a particular crash (or all crashes) and silence the health warning::

    `ceph crash archive <crash-id>`
    ceph crash archive-all

### 14.2.3

* The RGW `num_rados_handles` has been removed. If you were using a value of
  `num_rados_handles` greater than 1, multiply your current
  `objecter_inflight_ops` and `objecter_inflight_op_bytes` parameters by the
  old `num_rados_handles` to get the same throttle behavior.

### 13.2.7

MDS:

> Cache trimming is now throttled. Dropping the MDS cache via the ` "ceph tell mds.<foo> cache drop"` command or large reductions in the cache size will no longer cause service unavailability.



### 13.2.0


  * The monitor daemon uses significantly less disk space when undergoing
    recovery or rebalancing operations.

- *CephFS*:

  * Snapshots are now stable when combined with multiple MDS daemons.


# Openstack 的差异
对nfs的使用支持如何?

Wallaby对应Mimic-Pacific

rocky到2023.1后面那个版本

当前业务已升级到train版本, 局部还在N版本

## nova
### 27.0
### Zed

### Yoga

### Wallaby

-   The Hyper-V driver can now attach Cinder RBD volumes. The minimum requirements are Ceph 16 (Pacific) and Windows Server 2016.

-   The Hyper-V virt driver can now attach Cinder RBD volumes.

### Victoria
-   New `[glance]/enable_rbd_download` config option was introduced. The option allows for the configuration of direct downloads of Ceph hosted glance images into the libvirt image cache via rbd when `[glance]/enable_rbd_download= True` and `[glance]/rbd_user`, `[glance]/rbd_pool`, `[glance]/rbd_connect_timeout`, `[glance]/rbd_ceph_conf` are correctly configured.
-   Added params `[libvirt]/rbd_destroy_volume_retries`, defaulting to 12, and `[libvirt]/rbd_destroy_volume_retry_interval`, defaulting to 5, that Nova will use when trying to remove a volume from Ceph in a retry loop that combines these parameters together. Thus, maximum elapsing time is by default 60 seconds.
-   Nova tries to remove a volume from Ceph in a retry loop of 10 attempts at 1 second intervals, totaling 10 seconds overall - which, due to 30 second ceph watcher timeout, might result in intermittent object removal failures on Ceph side ([bug 1856845](https://bugs.launchpad.net/nova/+bug/1856845)). Setting default values for `[libvirt]/rbd_destroy_volume_retries` to 12 and `[libvirt]/rbd_destroy_volume_retry_interval` to 5, now gives Ceph reasonable amount of time to complete the operation successfully.

-   The libvirt RBD image backend module can now handle a Glance multistore environment where multiple RBD clusters are in use across a single Nova/Glance deployment, configured as independent Glance stores. In the case where an instance is booted with an image that does not exist in the RBD cluster that Nova is configured to use, Nova can ask Glance to copy the image from whatever store it is currently in to the one that represents its RBD cluster. To enable this feature, set `[libvirt]/images_rbd_glance_store_name` to tell Nova the Glance store name of the RBD cluster it uses.
-   Glance multistore configuration with multiple RBD backends is now supported within Nova for libvirt RBD-backed images using `[libvirt]/images_rbd_glance_store_name` configuration option.

### ussuri
-   Nova now has a config option called `[workarounds]/never_download_image_if_on_rbd` which helps to avoid pathological storage behavior with multiple ceph clusters. Currently, Nova does _not_ support multiple ceph clusters properly, but Glance can be configured with them. If an instance is booted from an image residing in a ceph cluster other than the one Nova knows about, it will silently download it from Glance and re-upload the image to the local ceph privately for that instance. Unlike the behavior you expect when configuring Nova and Glance for ceph, Nova will continue to do this over and over for the same image when subsequent instances are booted, consuming a large amount of storage unexpectedly. The new workaround option will cause Nova to refuse to do this download/upload behavior and instead fail the instance boot. It is simply a stop-gap effort to allow unsupported deployments with multiple ceph clusters from silently consuming large amounts of disk space.

### train
-   The reporting for bytes available for RBD has been enhanced to accomodate [unrecommended](http://docs.ceph.com/docs/luminous/start/hardware-recommendations/#hard-disk-drives) Ceph deployments where multiple OSDs are running on a single disk. The new reporting method takes the number of configured replicas into consideration when reporting bytes available.
-   The libvirt driver’s RBD imagebackend no longer supports setting force_raw_images to False. Setting force_raw_images = False and images_type = rbd in nova.conf will cause the nova compute service to refuse to start. To fix this, set force_raw_images = True. This change was required to fix the [bug 1816686](https://bugs.launchpad.net/nova/+bug/1816686).
    
    Note that non-raw cache image files will be removed if you set force_raw_images = True and images_type = rbd now.
    -   A new `[libvirt]/rbd_connect_timeout` configuration option has been introduced to limit the time spent waiting when connecting to a RBD cluster via the RADOS API. This timeout currently defaults to 5 seconds.
    
    This aims to address issues reported in [bug 1834048](https://bugs.launchpad.net/nova/+bug/1834048) where failures to initially connect to a RBD cluster left the nova-compute service inoperable due to constant RPC timeouts being hit.
### stein
-   Adds support for extending RBD attached volumes using the libvirt network volume driver.
### rocky
The `[workarounds]/ensure_libvirt_rbd_instance_dir_cleanup` configuration option has been introduced. This can be used by operators to ensure that instance directories are always removed during cleanup within the Libvirt driver while using `[libvirt]/images_type = rbd`. This works around known issues such as [bug 1414895](https://bugs.launchpad.net/nova/+bug/1414895) when cleaning up after an evacuation and [bug 1761062](https://bugs.launchpad.net/nova/+bug/1761062) when reverting from an instance resize.

Operators should be aware that this workaround only applies when using the libvirt compute driver and rbd images_type as enabled by the following configuration options:

-   `[DEFAULT]/compute_driver = libvirt`
    
-   `[libvirt]/images_type = rbd`
### queens
-   Nova now requires Ceph/librados >= 11.1.0 if running under Python 3 with the RBD image backend for the libvirt driver. Requirements for Python 2 users or users using a different backend remain unchanged.

QEMU 2.6.0 and Libvirt 2.2.0 allow LUKS encrypted RAW files, block devices and network devices (such as rbd) to be decrypted natively by QEMU. If qemu >= 2.6.0 and libvirt >= 2.2.0 are installed and the volume encryption provider is ‘luks’, the libvirt driver will use native QEMU decryption for encrypted volumes. The libvirt driver will generate a secret to hold the LUKS passphrase for unlocking the volume and the volume driver will use the secret to generate the required encryption XML for the disk. QEMU will then be able to read from and write to the encrypted disk natively, without the need of os-brick encryptors.

### Mitaka
-   When RBD is used for ephemeral disks and image storage, make snapshot use Ceph directly, and update Glance with the new location. In case of failure, it will gracefully fallback to the “generic” snapshot method. This requires changing the typical permissions for the Nova Ceph user (if using authx) to allow writing to the pool where vm images are stored, and it also requires configuring Glance to provide a v2 endpoint with direct_url support enabled (there are security implications to doing this). See [http://docs.ceph.com/docs/master/rbd/rbd-openstack/](http://docs.ceph.com/docs/master/rbd/rbd-openstack/) for more information on configuring OpenStack with RBD.

## cinder

### unreleased
-   [Bug #1997980](https://bugs.launchpad.net/cinder/+bug/1997980): RBD: Fixed failure to update rbd image features for multi-attach when features = 0.
### 2023.1
-   RBD driver: Sets the Ceph cluster FSID as the default value for the `rbd_secret_uuid` configuration option.
- -   RBD driver [bug #1960206](https://bugs.launchpad.net/cinder/+bug/1960206): Fixed `total_capacity` reported by the driver to the scheduler on Ceph clusters that have renamed the `bytes_used` field to `stored`. (e.g., [Nautilus](https://docs.ceph.com/en/nautilus/releases/nautilus/#upgrade-compatibility-notes)).
-   RBD Driver [bug #1957073](https://bugs.launchpad.net/cinder/+bug/1957073): Fixed snapshot deletion failure when its volume doesn’t exist.
### Zed
-   RBD driver [bug #1942210](https://bugs.launchpad.net/cinder/+bug/1942210): When creating a volume from a snapshot, the operation could fail due to an uncaught exception being raised during a check to see if the backend Ceph installation supported the clone v2 API. The driver now handles this situation gracefully.
	- librbd , 12就支持了. 但是kernel要3.11才支持, 所以我们的是应该没支持的.

### Yoga
-   When the Ceph backup driver is used for the backup service, restoring a backup to a volume created on a non-RBD backend fails. The cinder team has developed a fix but decided to do more thorough testing before including it in a release. When ready, the solution is expected to be backported to a future release in the Yoga series. The issue is being tracked as [Bug #1895035](https://bugs.launchpad.net/cinder/+bug/1895035).
-   **RBD driver: Enable Ceph V2 Clone API and Ceph Trash auto purge**
    
    In light of the fix for RBD driver [bug #1941815](https://bugs.launchpad.net/cinder/+bug/1941815), we want to bring the following information to your attention.
    
    Using the v2 clone format for cloned volumes allows volumes with dependent images to be moved to the trash - where they remain until purged - and allow the RBD driver to postpone the deletion until the volume has no dependent images. Configuring the trash purge is recommended to avoid wasting space with these trashed volumes. Since the Ceph Octopus release, the trash can be configured to automatically purge on a defined schedule. See the `rbd trash purge schedule` commands in the [rbd manpage](https://docs.ceph.com/en/octopus/man/8/rbd/).
-   RBD driver [bug #1941815](https://bugs.launchpad.net/cinder/+bug/1941815): Fixed deleting volumes with snapshots/volumes in the ceph trash space.
-   RBD driver [bug #1916843](https://bugs.launchpad.net/cinder/+bug/1916843): Fixed rpc timeout when backing up RBD snapshot. We no longer flatten temporary volumes and snapshots.
-   RBD driver [bug #1947518](https://bugs.launchpad.net/cinder/+bug/1947518): Corrected a regression caused by the fix for [Bug #1931004](https://bugs.launchpad.net/cinder/+bug/1931004) that was attempting to access the glance images RBD pool with write privileges when creating a volume from an image.
### Xena
-   [Bug #1931004](https://bugs.launchpad.net/cinder/+bug/1931004): Fixed use of incorrect stripe unit in RBD image clone causing volume-from-image to fail when using raw images backed by Ceph.

### Wallaby
-   Added new Ceph iSCSI driver rbd_iscsi. This new driver is derived from the rbd driver and allows all the same features as the rbd driver. The only difference is that volume attachments are done via iSCSI.

已知问题
-   **Anomalies with encrypted volumes**
    
    For the most part, users are happy with the cinder feature [Volume encryption supported by the key manager](https://docs.openstack.org/cinder/wallaby/configuration/block-storage/volume-encryption.html). There are, however, some edge cases that have revealed bugs that you and your users should be aware of.
    
    First, some background. The Block Storage API supports the creation of volumes in gibibyte (GiB) units. When a volume of a non-encrypted volume type of size _n_ is created, the volume contains _n_ GiB of usable space. When a volume of an encrypted type is requested, however, the volume contains less than _n_ GiB of usable space because the encryption metadata that must be stored within that volume in order for the volume to be usable consumes an amount of the otherwise usable space.
    
    Although the encryption metadata consumes less than 1% of the volume, suppose that a user wants to retype a volume of a non-encrypted type to an encrypted type of the same size. If the non-encrypted volume is “full”, we are in the position of trying to fit 101% of its capacity into the encrypted volume, which is not possible under the current laws of physics, and the retype should fail (see [Known Issues](https://docs.openstack.org/cinder/wallaby/configuration/block-storage/volume-encryption.html) for volume encryption in the cinder documentation).
    
    (Note that whether a volume should be considered “full”, even if it doesn’t contain exactly _n_ GiB of data for an _n_ GiB volume, can depend upon the storage backend technology used.)
    
    A similar situation can arise when a user creates a volume of an encrypted volume type from an image in Glance. If the image happens to be sized very close to the gibibyte boundary given by the requested volume size, the operation may fail if the image data plus the encryption metadata exceeds the requested volume size.
    
    So far, the behavior isn’t anomalous; it’s basically what you’d expect once you are aware that the encryption metadata must be stored in the volume and that it consumes some space.
    
    We recently became aware of the following anomalies, however, when using the current RBD driver with a Ceph storage backend.
    
    -   When creating an encrypted volume from an image in Glance that was created from a non-encrypted volume uploaded as an image, or an image that just happens to be sized very close to the gibibyte boundary given by the requested volume size, the space consumed by the encryption header may not leave sufficient space for the data contained in the image. In this case, the data is silently truncated to fit within the requested volume size.
        
    -   Similarly, when creating an encrypted volume from a snapshot of an encrypted volume, if the amount of data in the original volume at the time the snapshot was created is very close to the gibibyte boundary given by the volume’s size, it is possible for the data in the new volume to be silently truncated.
        
    
    Not to put too fine a point on it, silent truncation is worse than failure, and the Cinder team will be addressing these issues in the next release. Additionally (as if that isn’t bad enough!), we suspect that the above anomalies will also occur when using volume encryption with NFS-based storage backends, though this has not yet been reported or confirmed.
-   RBD driver: Prior to this release, the Cinder project did not have a statement concerning what versions of Ceph are supported by Cinder. We hereby announce that:
    
    -   For a given OpenStack release, Cinder supports the current Ceph active stable releases plus the two prior releases.
        
    -   For any OpenStack release, it is expected that the versions of the Ceph client and server are in alignment.
        
    
    The [Ceph RADOS Block Device (RBD)](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/ceph-rbd-volume-driver.html) driver documentation has been updated to reflect this policy and explains it in more detail.
    

-   Ceph/RBD volume backends will now assume exclusive cinder pools, as if they had `rbd_exclusive_cinder_pool = true` in their configuration.
    
    This helps deployments with a large number of volumes and prevent issues on deployments with a growing number of volumes at the small cost of a slightly less accurate stats being reported to the scheduler.
-   RBD driver [bug #1907964](https://bugs.launchpad.net/cinder/+bug/1907964): Add support for fast-diff on backup images stored in Ceph. Provided fast-diff is supported by the backend it will automatically be enabled and used. With fast-diff enabled, the generation of diffs between images and snapshots as well as determining the actual data usage of a snapshot is speed up significantly.
-   [Bug 1913449](https://bugs.launchpad.net/cinder/+bug/1913449): Fix RBD driver _update_volume_stats() failing when using Ceph Pacific python rados libraries. This failed because we were passing a str instead of bytes to cluster.mon_command()
-   Ceph/RBD: Fix cinder taking a long time to start for Ceph/RBD backends. ([Related-Bug #1704106](https://bugs.launchpad.net/cinder/+bug/1704106))
    

-   Ceph/RBD: Fix Cinder becoming non-responsive and stats gathering taking longer that its period. ([Related-Bug #1704106](https://bugs.launchpad.net/cinder/+bug/1704106))


-   **Supported Ceph versions**
    
    The Cinder project wishes to clarify its policy concerning what versions of Ceph are supported by Cinder.
    
    -   For a given OpenStack release, Cinder supports the current Ceph active stable releases plus the two prior releases.
        
    -   For any OpenStack release, it is expected that the versions of the Ceph client and server are in alignment.
        
    
    The [Ceph RADOS Block Device (RBD)](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/ceph-rbd-volume-driver.html) driver documentation has been updated to reflect this policy and explains it in more detail.
-   RBD driver [Bug #1898918](https://bugs.launchpad.net/cinder/+bug/1898918): Fix thread block caused by the flatten operation during cloning a volume. Now the flatten operation is executed in a different thread.
-   RBD driver [bug #1901241](https://bugs.launchpad.net/cinder/+bug/1901241): Fixed an issue where decreasing the `rbd_max_clone_depth` configuration option would prevent volumes that had already exceeded that depth from being cloned.

### Victoria

-   RBD driver: the `rbd_keyring_conf` configuration option, which was deprecated in the Ussuri release, has been removed. If it is present in a configuration file, its value will silently be ignored. For more information, see [OSSN-0085](https://wiki.openstack.org/wiki/OSSN/OSSN-0085): Cinder configuration option can leak secret key from Ceph backend.
-   [Bug #1828386](https://bugs.launchpad.net/cinder/+bug/1828386): Fix the bug that a volume retyped from another volume type to a replicated or multiattach type cannot have replication or multiattach enabled in rbd driver.
-   [Bug #1873738](https://bugs.launchpad.net/cinder/+bug/1873738): RBD Driver: Added cleanup for residue destination file if the copy image to encrypted volume operation fails.
### Ussuri
-   RBD driver: support added for reverting a volume to the most recent snapshot taken.
    
    Please be aware of the following known issues with this operation and the Ceph storage backend:
    
    -   Rolling back a volume to a snapshot overwrites the current volume with the data from the snapshot, and the time it takes to complete this operation increases with the size of the volume.
        
        It is faster to create a new volume from a snapshot. You may wish to recommend this option to your users whose use cases do not strictly require revert-to-snapshot.
        
    -   The efficiency of revert-to-snapshot is also dependent upon the Ceph storage backend in use, namely, whether or not BlueStore is being used in your Ceph installation.
        
    
    Please consult the Ceph documentation for details.
    -   Fix volume migration fails in the same ceph RBD pool. [Bug 1871524](https://bugs.launchpad.net/cinder/+bug/1871524).
### Train
-   Catch argument exceptions when configuring multiattach for rbd volumes. This allows multiattach images with flags already set to continue instead of raising an exception and failing.
-   Rbd replication secondary device could set different user and keyring with primary cluster. Secondary secret_uuid value is configed in libvirt secret, and libvirtd using secondary secret reconnect to secondary cluster after Cinder failover host.
-   Fixed issue where all Ceph RBD backups would be incremental after the first one. The driver now honors whether `--incremental` is specified or not.
### stein

-   RBD driver has added multiattach support. It should be noted that replication and multiattach are mutually exclusive, so a single RBD volume can only be configured to support one of these features at a time. Additionally, RBD image features are not preserved which prevents a volume being retyped from multiattach to another type. This limitation is temporary and will be addressed soon.
-   Add support for deferred deletion in the RBD volume driver.
## glance



除Newton版本全部没有cephfs/rbd/cephfs关键词日志





# RedHat Enterprise Storage 

[Red Hat Ceph Storage Life Cycle - Red Hat Customer Portal](https://access.redhat.com/support/policy/updates/ceph-storage)




### 6

应该是17版本

[Chapter 3. New features Red Hat Ceph Storage 6.0 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/6.0/html/release_notes/enhancements)


* **Compression on-wire with msgr2 protocol is now available**
* **`librbd` plugin named persistent write log cache to reduce latency**
	* With this release, the new `librbd` plugin named Persistent Write Log Cache (PWL) provides a persistent, fault-tolerant write-back cache targeted with SSD devices. It greatly reduces latency and also improves performance at low `io_depths`. This cache uses a log-ordered write-back design which maintains checkpoints internally, so that writes that get flushed back to the cluster are always crash consistent. Even if the client cache is lost entirely, the disk image is still consistent; but the data will appear to be stale.
	* 意思是

这个是在quincy版本才支持的. 所以这个版本还是太新了.
* - The allocation metadata is removed from RocksDB and now performs a full destage of the allocator object with the OSD allocation.
- With cache age binning, older onodes might be assigned a lower priority than the hot workload data. See the [_Ceph BlueStore_](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/6/html-single/administration_guide/#ceph-bluestore_admin) for more details.



不看6了...


### 5

应该是15或者16


只支持容器化集群

bluestore支持了4K粒度,
With this release, the default value of BlueStore’s `min_alloc_size` for SSDs and HDDs is 4 KB. This enables better use of space with no impact on performance.

**Sharding of RocksDB database using column families is supported**

rbd

**Improved librbd small I/O performance**

Previously, in an NVMe based Ceph cluster, there were limitations in the internal threading architecture resulting in a single librbd client struggling to achieve more than 20K 4KiB IOPS.

With this release, librbd is switched to an asynchronous reactor model on top of the new ASIO-based neorados API thereby increasing the small I/O throughput potentially by several folds and reducing latency.


**Built in schedule for purging expired RBD images**

Previously, the storage administrator could set up a cron-like job for the `rbd trash purge` command.

With this release, the built-in schedule is now available for purging expired RBD images. The `rbd trash purge schedule add` and the related commands can be used to configure the RBD trash to automatically purge expired images based on a defined schedule.

See the [_Defining an automatic trash purge schedule_](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/5/html-single/block_device_guide/#defining-an-automatic-trash-purge-schedule_block) section in the _Red Hat Ceph Storage Block Device Guide_ for more information.

支持内建的trash周期清理等逻辑.


**Overriding read-from-replica policy in librbd clients is supported**


**Online re-sparsification of RBD images**

Previously, reclaiming space for image extents that are zeroed and yet fully allocated in the underlying OSD object store was highly cumbersome and error prone. With this release, the new `rbd sparsify` command can now be used to scan the image for chunks of zero data and deallocate the corresponding ranges in the underlying OSD object store.


**ocf:ceph:rbd cluster resource agent supports namespaces**

Previously, it was not possible to use ocf:ceph:rbd cluster resource agent for images that exist within a namespace.

With this release, the new `pool_namespace` resource agent parameter can be used to handle images within the namespace.


**RBD images can be imported instantaneously**

With the `rbd import` command, the new image becomes available for use only after it is fully populated.

With this release, the image live-migration feature is extended to support external data sources and can be used as an alternative to `rbd import`. The new image can be linked to local files, remote files served over HTTP(S) or remote Amazon S3-compatible buckets in `raw`, `qcow` or `qcow2` formats and becomes available for use immediately. The image is populated as a background operation which can be run while it is in active use.


**Snapshot-based mirroring of RBD images**

The journal-based mirroring provides fine-grained crash-consistent replication at the cost of double-write penalty where every update to the image is first recorded to the associated journal before modifying the actual image.

With this release, in addition to journal-based mirroring, snapshot-based mirroring is supported. It provides coarse-grained crash-consistent replication where the image is mirrored using the mirror snapshots which can be created manually or periodically with a defined schedule. This is supported by all clients and requires a less stringent recovery point objective (RPO).



### 4

看起来是13

bluestore稳定

**Asynchronous recovery for non-acting OSD sets**
Previously, recovery with Ceph was a synchronous process by blocking write operations to objects until those objects were recovered. In this release, the recovery process is now asynchronous by not blocking write operations to objects only in the non-acting set of OSDs. This new feature requires having more than the minimum number of replicas, as to have enough OSDs in the non-acting set.

The new configuration option, `osd_async_recovery_min_cost`, controls how much asynchronous recovery to do. The default value for this option is `100`. A higher value means asynchronous recovery will be less, whereas a lower value means asynchronous recovery will be more.


**Introduction of `diskprediction` module**

**Erasure coding for Ceph Block Device**

**RBD performance monitoring and metrics gathering tools**

**Cloned images can be created from non-primary images**

Creating cloned child RBD images from mirrored non-primary parent image is now supported. Previously, cloning of mirrored images was only supported for primary images. When cloning golden images for virtual machines, this restriction prevented the creation of new cloned images from the golden non-primary image. This update removes this restriction, and cloned images can be created from non-primary mirrored images.


**Segregating RBD images within isolated namespaces within the same pool**

RBD images can now be segregated within isolated namespaces within the same pool. When using Ceph Block Devices directly without a higher-level system, such as OpenStack or OpenShift Container Storage, it was not possible to restrict user access to specific RBD images. When combined with CephX capabilities, users can be restricted to specific pool namespaces to restrict access to RBD images.

**Moving RBD images between different pools within the same cluster**

This version of Red Hat Ceph Storage adds the ability to move RBD images between different pools within the same cluster. For details, see the [_Moving images between pools_](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/4/html-single/block_device_guide/#moving-images-between-pools_block) section in the _Block Device Guide_ for Red Hat Ceph Storage 4.

**Long-running RBD operations can run in the background**

Long-running RBD operations, such as image removal or cloned image flattening, can now be scheduled to run in the background. RBD operations that involve iterating over every backing RADOS object for the image can take a long time depending on the size of the image. When using the CLI to perform one of these operations, the `rbd` CLI is blocked until the operation is complete. These operations can now be scheduled to run by the Ceph Manager as a background task by using the `ceph rbd task add` commands. The progress of these tasks is visible on the Ceph dashboard as well as by using the CLI.


### 3
Red Hat Ceph Storage 3 这个比较像是12

[Chapter 3. New features Red Hat Ceph Storage 3.3 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3.3/html/release_notes/enhancements)


ceph-ansible


**The new `device_class` Ansible configuration option**

With the ``device_class`feature, you can alleviate post deployment configuration by updating the `groups_vars/osd.yml`` file in the desired layout. This feature offers you multi-backend support by avoiding to comment out sections after deploying Red Hat Ceph Storage.


**The default BlueStore and BlueFS allocator is now `bitmap`**


**New `omap` usage statistics per PG and OSD**

**Listing RADOS objects in a specific PG**
 