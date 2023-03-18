---
title: qemu之drive-mirror
subtitle: tech
date: 2022-11-18 21:28:31
updated:
tags: [qemu, ceph, rbd]
categories: [专业]
---



## bitmap[^Qemu 中 Bitmap 的应用 – 滴滴云博客]



[Qemu 中 Bitmap 的应用 – 滴滴云博客](https://blog.didiyun.com/index.php/2018/12/05/qemu-bitmap/)

根据实验, 我源卷以8k, 200 iops, 最后落到目标端是128K, 64K的请求.

这个写是否是同步的呢?

bdrv_get_dirty_count


``` c
    /* Find the number of consective dirty chunks following the first dirty
     * one, and wait for in flight requests in them. */
    bdrv_dirty_bitmap_lock(s->dirty_bitmap);
    while (nb_chunks * s->granularity < s->buf_size) {
        int64_t next_dirty;
        int64_t next_offset = offset + nb_chunks * s->granularity;
        int64_t next_chunk = next_offset / s->granularity;
        if (next_offset >= s->bdev_length ||
            !bdrv_dirty_bitmap_get_locked(s->dirty_bitmap, next_offset)) {
            break;
        }
        if (test_bit(next_chunk, s->in_flight_bitmap)) {
            break;
        }

        next_dirty = bdrv_dirty_iter_next(s->dbi);
        if (next_dirty > next_offset || next_dirty < 0) {
            /* The bitmap iterator's cache is stale, refresh it */
            bdrv_set_dirty_iter(s->dbi, next_offset);
            next_dirty = bdrv_dirty_iter_next(s->dbi);
        }
        assert(next_dirty == next_offset);
        nb_chunks++;
    }
```


## bitmap 持久化存储 

The dirty bitmap can be used as follows for storage migration. To start migration:

blockdev-dirty-enable ide0-hd0 /var/lib/libvirt/dirty/diskname
management notes existence of dirty bitmap for /mnt/src/diskname.img in its private data
drive-mirror ide0-hd0 /mnt/dest/diskname.img
management notes /mnt/dest/diskname.img as the mirroring target in its private data
At this point, mirroring has taken a reference to the dirty bitmap.
To end migration:
blockdev-dirty-disable ide0-hd0
block-job-complete ide0-hd0
The dirty bitmap remains enabled until the BLOCK_JOB_COMPLETED event is sent.
When management receives the BLOCK_JOB_COMPLETED event, it notes switch to /mnt/dest/diskname.img (without dirty bitmap nor mirroring target) in its private data.
If management crashes between (6) and (7), it can examine the dirty bitmap on disk. If it is all-zeros, management can restart the virtual machine with /mnt/dest/diskname.img. If it has even a single zero bit, management can restart the virtual machine with the persistent dirty bitmap enabled, and later issue again a drive-mirror command (with sync='dirty') to restart from step 4.


### 默认的granularity 和buf-size

```
#define MAX_IN_FLIGHT 16
#define MAX_IO_BYTES (1 << 20) /* 1 Mb */
#define DEFAULT_MIRROR_BUF_SIZE (MAX_IN_FLIGHT * MAX_IO_BYTES)


...

    if (buf_size == 0) {
        buf_size = DEFAULT_MIRROR_BUF_SIZE;
    }


```


```
/**
 * Chooses a default granularity based on the existing cluster size,
 * but clamped between [4K, 64K]. Defaults to 64K in the case that there
 * is no cluster size information available.
 */
uint32_t bdrv_get_default_bitmap_granularity(BlockDriverState *bs)
{
    BlockDriverInfo bdi;
    uint32_t granularity;

    if (bdrv_get_info(bs, &bdi) >= 0 && bdi.cluster_size > 0) {
        granularity = MAX(4096, bdi.cluster_size);
        granularity = MIN(65536, granularity);
    } else {
        granularity = 65536;
    }

    return granularity;
}
```




```
mirror_start_job
...
    s->granularity = granularity;
    s->buf_size = ROUND_UP(buf_size, granularity);
```


## Q

### blockcopy和drive-mirror差别?


### 能否运行时修改drive-mirror的任务?

block-job-set-speed 有这个可以用.


## 接口
``` bash
暂停命令： virsh qemu-monitor-command 虚拟机uuid --pretty '{ "execute": "block-job-pause","arguments": { "device": "drive-virtio-disk0"} }'

恢复暂停任务命令：virsh qemu-monitor-command uuid --pretty '{ "execute": "block-job-resume","arguments": { "device": "drive-virtio-disk0"} }'

取消任务命令：virsh qemu-monitor-command 虚拟机uuid '{ "execute": "block-job-cancel", "arguments": { "device": "drive-virtio-disk0", "force": true } }'

执行备份命令：virsh qemu-monitor-command 虚拟机uuid  '{ "execute" : "drive-mirror" , "arguments" :{ "device" : "drive-virtio-disk0" , "sync" : "full" , "format": "raw","target" : "rbd:ceph/xxx_img" } }'

查询任务命令：virsh qemu-monitor-command DOMAIN --pretty '{ "execute": "query-block-jobs" }'

```


