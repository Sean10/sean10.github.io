---
title: ceph之clion的cmake.md
subtitle: cmake
date: 2020-10-17 14:43:31
updated:
tags: [ceph, 编译]
categories: [专业]
---


# 调查

按照一般的编译过程, 是可以在`CentOS`或者`Debian`系的环境里直接执行`do_cmake.sh`然后自动安装`install-deps.sh`, 然后关闭一堆功能触发`cmake`的. 

不过因为暂时没找到可以进行环境隔离比如通过在`docker`内编译ceph(初步尝试, deps中出现了systemd, 而docker内使用使用的一个fake-systemd, 这俩产生了冲突, 不知道有没有专门针对docker做的一个fake版本), 从而绕过弄坏`OSX`宿主机的依赖的风险的. 在这个安装依赖的脚本中是有针对`freeBSD`的`pkg`安装命令的, 但是和目前主流推荐的`brew`还是不一样的, `pkg`还是给服务端使用为主. 怕产生污染, 所以我暂时选择手动来处理这个依赖的问题.

当然, 如果有远程服务器, 可以利用`remote develop`, 通过服务器编译也能把成功建立索引.

# docker调试用运行
## vstart.sh

## ceph/daemon[^9]
根据[^11]的`image`的`build`仓库来看, 这个docker也是可以来触发编译的?

> make CEPH_DEVEL=true FLAVORS="luminous-12.2.13,centos,7" build
单看这个脚本, 看起来可以编出来的样子.

> make CEPH_DEVEL=true FLAVORS="nautilus,centos,7" build
这里为什么这个版本的centos里装的是真实的systemd?, 我之前拿官方的centos的image就是`fake-systemd`呢... 搜了下, 用`fakesystemd`是为了`cgroup`与宿主机的隔离. 手动卸载再安装其实也是可以的.

不对, 这个镜像里是直接配置的`yum install -y ceph-mon`等...不是编译出来的...

## 自制Docker调试
``` bash
 docker run -itd --name centos-test centos:centos7
```

## opensuse ceph-dev-docker
[^12], 这个是`opensuse`的.

``` bash
#src.rpm 12.2.13
docker run -itd \
  -v /Users/sean10/Code/ceph/ceph-12.2.13:/ceph \
  -v /Users/sean10/Code/ceph/ceph-ccache-12.2.13:/root/.ccache \
  -v /Users/sean10/Code/ceph-dev-docker/shared:/shared \
  --net=host \
  --name=ceph-dev \
  --hostname=ceph-dev \
  --add-host=ceph-dev:127.0.0.1 \
  ceph-dev-docker
# luminous版本
docker run -itd \
  -v /Users/sean10/Code/ceph/ceph-14.2.9:/ceph \
  -v /Users/sean10/Code/ceph/ceph-ccache-14.2.9:/root/.ccache \
  -v /Users/sean10/Code/ceph-dev-docker/shared:/shared \
  --net=host \
  --name=ceph-dev \
  --hostname=ceph-dev \
  --add-host=ceph-dev:127.0.0.1 \
  ceph-dev-docker
# nautilus
docker run -itd \
  -v /Users/sean10/Code/ceph/master/ceph:/ceph \
  -v /Users/sean10/Code/ceph/ceph-ccache-branch-luminous:/root/.ccache \
  -v /Users/sean10/Code/ceph-dev-docker/shared:/shared \
  --net=host \
  --name=ceph-dev \
  --hostname=ceph-dev \
  --add-host=ceph-dev:127.0.0.1 \
  ceph-dev-docker


docker attach ceph-dev

NPROC=4 ./setup-ceph.sh
docker exec -it ceph-dev /bin/zsh
```

通过传给`cmake`的`CMAKE_BUILD_TYPE`来确认编译出来的是否携带debuginfo.[^13]

> Release —— 不可以打断点调试，程序开发完成后发行使用的版本，占的体积小。 它对代码做了优化，因此速度会非常快，
> 在编译器中使用命令： -O3 -DNDEBUG 可选择此版本。
> Debug ——调试的版本，体积大。
> 在编译器中使用命令： -g 可选择此版本。
> MinSizeRel—— 最小体积版本
> 在编译器中使用命令：-Os -DNDEBUG可选择此版本。
> RelWithDebInfo—— 既优化又能调试。
> 在编译器中使用命令：-O2 -g -DNDEBUG可选择此版本。

默认是`RelWithDebInfo`, 我给他传了俩参数` -DWITH_LTTNG=ON -DDWITH_BABELTRACE=ON`

### python-devel not found.
结论: 看上去是`suse`的`tumbleweed`现在的源里没`python2`的包了[^14]? 而这个仓库里的`master`分支如果编`ceph`还没去除`python2`的版本, 就会有问题.

根据这个来看[Re: \[opensuse\-factory\] Removal of Python2 from openSUSE Tumbleweed](https://lists.opensuse.org/opensuse-factory/2020-01/msg00126.html)

高版本的suse把python2彻底抛弃了, 但是是不是也有办法加回来安装呢? 搜了一圈, 主流的思路是这种大版本迭代, 如果始终要考虑老版本兼容性, 始终去兼容`python2`, 对于开发的迭代并不利, 不如让开源软件去遵循这个规则, 统一进行大版本升级. 这种在商业产品里, 做法已经比较成熟了, 都是软件去适配系统. 只是`linux`系统在`python`以前可能没依赖性这么强的类似基础框架组件性质的部分, 所以对于彻底不兼容的软件会相对少一点. hhh, 说到这个, 这种换到公司里的代码开发, 就是为了兼容性, 当单元测试少时, 就不重构, 变成垃圾代码. 而开源的, 就看开发者的能力了. 好像见到的一般都有大神在把控代码质量的~

所以理论上其实ceph应该不完全依赖python2了. 我应该通过其他方式去判断是否要编python2相关.

根据不同版本的`ceph`的`install-deps.sh`和`ceph.spec`, 可知`mimic`和`nautilus`的版本还没有完全可以免去`python2.7`, 而`Octopus`里彻底去除了`python2`的编译.

#### tmubleweed 的repo路径
Factory 和 Tumbleweed 合并了, 所以直接使用 factory 的源就好. 不过大多数镜像只提供了 repo-oss 和 repo-non-oss 这两个源

https://download.opensuse.org/repositories/openSUSE:Factory/standard/openSUSE:Factory.repo 
搜出来显示这个,但是这个路径实际上已经不存在了...不知道怎么给他们提

[Install package devel:languages:python:Factory / python](https://software.opensuse.org/download.html?project=devel%3Alanguages%3Apython%3AFactory&package=python)
[Install package devel:languages:python / python\-virtualenv](https://software.opensuse.org/download.html?project=devel%3Alanguages%3Apython&package=python-virtualenv)

[openSUSE:Build Service 仓库详解 \- openSUSE Wiki](https://zh.opensuse.org/openSUSE:Build_Service_%E4%BB%93%E5%BA%93%E8%AF%A6%E8%A7%A3)

看上去这上面源里的是src.rpm. 咋Search能看到, 就是提示转不上呢?
我先配置上面这个, 给装上去了.

#### 某些疑似镜像源中似乎还有python2的库
[Index of /pub/opensuse/tumbleweed/repo/oss/x86\_64/](https://ftp.lysator.liu.se/pub/opensuse/tumbleweed/repo/oss/x86_64/?C=M&O=A)

但是下面这个源里明明也是oss, 但是却有呢?





### 稳定的Leap 15版本的nautilus编译
```
export NAME=ceph-dev-nautilus
export VERSION=nautilus
export CEPH=/Users/sean10/Code/ceph/ceph-14.2.9
export CCACHE=/Users/sean10/Code/ceph/ceph-ccache-14.2.9
cd /share/bin/nautilus
bash -x ./setup.sh

```
这个倒是安装成功, 开始触发编译了.

不过中间`dashboard`的`frontend`在安装pip时报了点`Input/Output Error`, 暂时不是重点, 在`setup-ceph.sh`里设置`-DWITH_MGR_DASHBOARD_FRONTEND=OFF`, 就可以通过编译了. 就是本地编译有点慢...

``` bash
cd /ceph/src
OSD=3 MON=3 MGR=1 CEPH_BUILD_ROOT=/ceph/build bash -x ./vstart.sh -n 
```

#### ccache make -j8 ceph-mon
> [ 97%] Building CXX object src/mon/CMakeFiles/mon.dir/AuthMonitor.cc.o
> c++: internal compiler error: Killed (program cc1plus)
> Please submit a full bug report,
> with preprocessed source if appropriate.
> See <https://bugs.opensuse.org/> for instructions.
> make[3]: *** [src/mon/CMakeFiles/mon.dir/build.make:351: src/mon/CMakeFiles/mon.dir/MonmapMonitor.cc.o] Error 4

看着跟`OOM`的表现有点像.

根据[linux \- make \-j 8 g\+\+: internal compiler error: Killed \(program cc1plus\) \- Stack Overflow](https://stackoverflow.com/questions/30887143/make-j-8-g-internal-compiler-error-killed-program-cc1plus)这篇说的, 可能是我开的线程太多, 给他分配的内存又没那么多引起的? 我试试

降低以后的确编过了.

#### vstart.sh erasure-code
> load dlopen(/ceph/build/lib/erasure-code/libec_jerasure.so): /ceph/build/lib/erasure-code/libec_jerasure.so: cannot open shared object file: No such file or directory

``` bash
╭─root@ceph-dev-nautilus /ceph 
╰─# cat src/vstart.sh | grep erasure-code  
        [ -z "$EC_PATH" ] && EC_PATH=$CEPH_LIB/erasure-code
```
暂时没找到怎么只是make , 生成到`build/lib/erasure-code`目录下的功能.


#### vstart.sh  ERROR: error creating empty object store in /data/ceph/build/dev/osd0: (22) Invalid argument

根据[Support \#23433: Ceph cluster doesn't start \- ERROR: error creating empty object store in /data/ceph/build/dev/osd0: \(22\) Invalid argument \- bluestore \- Ceph](https://tracker.ceph.com/issues/23433)这里的回复, 看起来这个是因为毕竟是虚拟化的, 所以指向的/dev/目录下的内容并不是块设备引起的吧?

# 实践
采用官方给的关闭的大部分配置文件的方案[^1], 可以避过大部分坑, 至少`Luminous`版本做到了. 

## Docker编译
根据[^7], 还有有参考价值的.

## 官方CI[^10]
似乎这里主要运行`test`和`image`, 那个`dev`的镜像看起来是给调试镜像版本用的, 并不是直接提供运行`vstart.sh`来调试编译版本的.


## Luminous
### 最后按照下述方案处理好软链和安装包之后的cmake选项
```
-DCMAKE_C_COMPILER=/usr/bin/clang   -DCMAKE_CXX_COMPILER=/usr/bin/clang++   -DCMAKE_EXE_LINKER_FLAGS="-L/usr/local/opt/llvm/lib"   -DENABLE_GIT_VERSION=OFF   -DSNAPPY_ROOT_DIR=/usr/local/Cellar/snappy/1.1.7_1   -DWITH_BABELTRACE=OFF   -DWITH_BLUESTORE=OFF   -DWITH_CCACHE=OFF   -DWITH_CEPHFS=OFF   -DWITH_KRBD=OFF   -DWITH_LIBCEPHFS=OFF   -DWITH_LTTNG=OFF   -DWITH_LZ4=OFF   -DWITH_MANPAGE=ON   -DWITH_MGR=OFF   -DWITH_MGR_DASHBOARD_FRONTEND=OFF   -DWITH_RADOSGW=OFF   -DWITH_RDMA=OFF   -DWITH_SPDK=OFF   -DWITH_SYSTEMD=OFF   -DWITH_TESTS=OFF  -DWITH_XFS=OFF -DCMAKE_C_COMPILER=/usr/bin/clang   -DCMAKE_CXX_COMPILER=/usr/bin/clang++   -DCMAKE_EXE_LINKER_FLAGS="-L/usr/local/opt/llvm/lib"   -DENABLE_GIT_VERSION=OFF   -DSNAPPY_ROOT_DIR=/usr/local/Cellar/snappy/1.1.7_1   -DWITH_BABELTRACE=OFF   -DWITH_BLUESTORE=OFF   -DWITH_CCACHE=OFF   -DWITH_CEPHFS=OFF   -DWITH_KRBD=OFF   -DWITH_LIBCEPHFS=OFF   -DWITH_LTTNG=OFF   -DWITH_LZ4=OFF   -DWITH_MANPAGE=ON   -DWITH_MGR=OFF   -DWITH_MGR_DASHBOARD_FRONTEND=OFF   -DWITH_RADOSGW=OFF   -DWITH_RDMA=OFF   -DWITH_SPDK=OFF   -DWITH_SYSTEMD=OFF   -DWITH_TESTS=OFF  -DWITH_XFS=OFF -DCMAKE_EXPORT_COMPILE_COMMANDS=on  -DPYTHON_LIBRARY=$(python-config --prefix)/lib/libpython2.7.dylib -DPYTHON_INCLUDE_DIR=$(python-config --prefix)/include/python2.7 -DPYTHON3_LIBRARY=$(python3-config --prefix)/lib/libpython3.9.dylib -DPYTHON3_INCLUDE_DIR=$(python3-config --prefix)/include/python3.9 -DOPENSSL_ROOT_DIR=/usr/local/Cellar/openssl@1.1/1.1.1o/
```



### 官方文档方案


``` bash
brew install llvm
brew install snappy ccache cmake pkg-config
pip install cython
pip3 install cython
brew cask install osxfuse
mkdir build
cd build
export PKG_CONFIG_PATH=/usr/local/Cellar/nss/3.48/lib/pkgconfig:/usr/local/Cellar/openssl/1.0.2t/lib/pkgconfig
cmake .. -DBOOST_J=4 \
  -DCMAKE_C_COMPILER=/usr/local/opt/llvm/bin/clang \
  -DCMAKE_CXX_COMPILER=/usr/local/opt/llvm/bin/clang++ \
  -DCMAKE_EXE_LINKER_FLAGS="-L/usr/local/opt/llvm/lib" \
  -DENABLE_GIT_VERSION=OFF \
  -DSNAPPY_ROOT_DIR=/usr/local/Cellar/snappy/1.1.7_1 \
  -DWITH_BABELTRACE=OFF \
  -DWITH_BLUESTORE=OFF \
  -DWITH_CCACHE=OFF \
  -DWITH_CEPHFS=OFF \
  -DWITH_KRBD=OFF \
  -DWITH_LIBCEPHFS=OFF \
  -DWITH_LTTNG=OFF \
  -DWITH_LZ4=OFF \
  -DWITH_MANPAGE=ON \
  -DWITH_MGR=OFF \
  -DWITH_MGR_DASHBOARD_FRONTEND=OFF \
  -DWITH_RADOSGW=OFF \
  -DWITH_RDMA=OFF \
  -DWITH_SPDK=OFF \
  -DWITH_SYSTEMD=OFF \
  -DWITH_TESTS=OFF \
  -DWITH_XFS=OFF
```
由于我的主要目的是在`CLion`中使用, 所以我在`Perference->Build,Execution,Deployment->Cmake->Cmake Options`中添加的是上面的配置的单行形式.

### 出现的报错
#### BUILD NSS_INCLUDE_DIRS: NSS_INCLUDE_DIR-NOTFOUND
查看`find_package(NSS)`中调用的其实是`pkg-config`来查找`nss.pc`, 使用`pkg-config --cflags --libs nss`报不存在, 所以说明`pkg-config`的扫描路径中没有上面这个`pc`文件. 

通过`brew list nss`查找到`nss.pc`文件所在, 通过`pkg-config --variable pc_path pkg-config`找到`pc`的默认搜索路径, 有两种方案
1. 做个软链到`pkg-config`的默认搜索路径中
``` bash
ln -s /usr/local/Cellar/nss/3.35/lib/pkgconfig/nss.pc  /usr/local/lib/pkgconfig/nss.pc 
```
2. `pkg_config`使用的搜索环境变量`PKG_CONFIG_PATH`中导入
``` bash
export PKG_CONFIG_PATH=/usr/local/Cellar/nss/3.48/lib/pkgconfig:/usr/local/Cellar/
```

我采用的第一种方案. 同理,还会出现`nspr`等需要这样处理的库.

#### NOT find PythonLibs: Found unsuitable version "2.7.10", but required[^3]
在CMake选项中增加指定下述的python库和头文件路径
``` cmake
-DPYTHON_LIBRARY=$(python-config --prefix)/lib/libpython2.7.dylib -DPYTHON_INCLUDE_DIR=$(python-config --prefix)/include/python2.7
```

#### TestBigEndian.cmake:49 (message): no suitable type found
暂时看到的资料都是说需要看测试代码和修改cmake文件的样子, 我并不需要编译, 只是需要建立索引, 所以我暂时注释了这部分的检查[^4,5,6]


## Nautilus
采用同样的方案, 暂时未去推进.

### 遇到的问题
#### findboost unknown compiler: AppleClang
报了一个暂时没去尝试解决的错误, 识别不了`clang`? 


## master

```
export PKG_CONFIG_PATH=/usr/local/Cellar/nss/3.35/lib/pkgconfig:/usr/local/Cellar/openssl@1.1/1.1.1o/lib/pkgconfig
```

把这里unknown compiler这段给屏蔽掉, 试试.
```
  if(CMAKE_CXX_COMPILER_ID STREQUAL GNU)
    set(toolset gcc)
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL Clang)
    set(toolset clang)
  else()
  # 换成这个
    set(toolset clang)
  #   message(SEND_ERROR "unknown compiler: ${CMAKE_CXX_COMPILER_ID}")
  endif()
  ```


```
cmake -DCMAKE_C_COMPILER=/usr/bin/clang   -DCMAKE_CXX_COMPILER=/usr/bin/clang++   -DCMAKE_EXE_LINKER_FLAGS="-L/usr/local/opt/llvm/lib"   -DENABLE_GIT_VERSION=OFF   -DSNAPPY_ROOT_DIR=/usr/local/Cellar/snappy/1.1.7_1   -DWITH_BABELTRACE=OFF   -DWITH_BLUESTORE=OFF   -DWITH_CCACHE=OFF   -DWITH_CEPHFS=OFF   -DWITH_KRBD=OFF   -DWITH_LIBCEPHFS=OFF   -DWITH_LTTNG=OFF   -DWITH_LZ4=OFF   -DWITH_MANPAGE=ON   -DWITH_MGR=OFF   -DWITH_MGR_DASHBOARD_FRONTEND=OFF   -DWITH_RADOSGW=OFF   -DWITH_RDMA=OFF   -DWITH_SPDK=OFF   -DWITH_SYSTEMD=OFF   -DWITH_TESTS=OFF  -DWITH_XFS=OFF -DCMAKE_C_COMPILER=/usr/bin/clang   -DCMAKE_CXX_COMPILER=/usr/bin/clang++   -DCMAKE_EXE_LINKER_FLAGS="-L/usr/local/opt/llvm/lib"   -DENABLE_GIT_VERSION=OFF   -DSNAPPY_ROOT_DIR=/usr/local/Cellar/snappy/1.1.7_1   -DWITH_BABELTRACE=OFF   -DWITH_BLUESTORE=OFF   -DWITH_CCACHE=OFF   -DWITH_CEPHFS=OFF   -DWITH_KRBD=OFF   -DWITH_LIBCEPHFS=OFF   -DWITH_LTTNG=OFF   -DWITH_LZ4=OFF   -DWITH_MANPAGE=ON   -DWITH_MGR=OFF   -DWITH_MGR_DASHBOARD_FRONTEND=OFF   -DWITH_RADOSGW=OFF   -DWITH_RDMA=OFF   -DWITH_SPDK=OFF   -DWITH_SYSTEMD=OFF   -DWITH_TESTS=OFF  -DWITH_XFS=OFF -DCMAKE_EXPORT_COMPILE_COMMANDS=on  -DPYTHON_LIBRARY=$(python-config --prefix)/lib/libpython2.7.dylib -DPYTHON_INCLUDE_DIR=$(python-config --prefix)/include/python2.7 -DPYTHON3_LIBRARY=$(python3-config --prefix)/lib/libpython3.9.dylib -DPYTHON3_INCLUDE_DIR=$(python3-config --prefix)/include/python3.9 -DOPENSSL_ROOT_DIR=/usr/local/Cellar/openssl@1.1/1.1.1o/ ..
```

用来索引好像还是用的.

## 调试增加选项
```
./configure CFLAGS='-g3 -O0' CXXFLAGS='-g3 -O0'.
```


## github版本cmake报缺失rocksdb和zstd
git里面有一些submodule, 这部分我没能拖回来, 导致这些目录是空的, 静态编译没能启动.

* rocksdb
* zstd
* civetweb
* dpdk
* googletest
* isa-l
* lua
* rapidjson
* spdk
* xxHash

# todo

> 问题并不是真正的CMakeLists.txt. CLion解析了cmake中引用的所有源文件,以启用大多数功能(导航,code-completion,重构).根据我的经验,索引大型项目可以花费几分钟(十分钟).

> 减轻这个问题的一种方法是标记项目的“第三方”目录:右键单击您的common目录和Mark directory as... > Libraries.如果需要,您甚至可以将目录排除在项目之外.
> 
> 还请注意,CLion索引的结果被缓存:在初始索引之后,即使在重新启动项目时,只应修复经修改的文件(注意,修改CMakeLists中的构建选项可能触发完整的reindex)


索引代码理论上只需要选择这些代码文件就可以, 但是实际上像ceph如果我cmake工程没能成功编译,他也是并没有把函数的调用关系给分析出来的. 哎, 静态分析功能还是得依赖clang.

* 根据CI里的log来看, 的确只是yum install ceph这种形式直接安装成果物构造image. 那src.rpm编译出成果物是在哪个jenkins里做的呢?

# Reference
1. [build on MacOS — Ceph Documentation](https://docs.ceph.com/en/latest/dev/macos/)
2. [pkg\-config 路径问题 \- 采男孩的小蘑菇 \- 博客园](https://www.cnblogs.com/flyinggod/p/12465966.html)
3. [CMake finding Python library and Python interpreter mismatch during pybind11 build · Issue \#99 · pybind/pybind11](https://github.com/pybind/pybind11/issues/99)
4. [Re: Weird CMake failures building 10\.4 on MacOS High Sierra \(10\.13\)](https://hypernews.slac.stanford.edu/HyperNews/geant4/get/installconfig/1889/1.html%3Finline=1.html)
5. [TestBigEndian\.cmake:51 \(messag no suitable type found \(\#16283\) · Issues · CMake / CMake · GitLab](https://gitlab.kitware.com/cmake/cmake/-/issues/16283)
6. [windows \- CMake internal error \(TEST\_BIG\_ENDIAN\) \- Stack Overflow](https://stackoverflow.com/questions/13255491/cmake-internal-error-test-big-endian)
7. [dockerize ceph集群和shell脚本编程 \- 知乎](https://zhuanlan.zhihu.com/p/98625873)
8. [CEPH CRUSH algorithm source code analysis](http://www.shalandis.com/original/2016/05/19/CEPH-CRUSH-algorithm-source-code-analysis/)
9. [ceph/daemon \- Docker Hub](https://hub.docker.com/r/ceph/daemon)
10. [Quay Container Registry · Quay](https://quay.io/repository/ceph-ci/ceph/manifest/sha256:7d170b95bd9861be7279a1d7cda1c3a5dd39469ebc5ff94c84e5c4e1ebbce42d)
11. [ceph/ceph\-container: Docker files and images to run Ceph in containers](https://github.com/ceph/ceph-container)
12. [ricardoasmarques/ceph\-dev\-docker](https://github.com/ricardoasmarques/ceph-dev-docker)
13. [Build Ceph — Ceph Documentation](https://docs.ceph.com/en/latest/install/build-ceph/)
14. [TUMBLEWEED Cant install python\-lxml Opensuse Tumbleweed](https://forums.opensuse.org/showthread.php/540286-Cant-install-python-lxml-Opensuse-Tumbleweed)
15. [ceph luminous版本编译及部署 \| itocm\.com](http://www.itocm.com/a/127B85AF17F747AE9C864B7246E91248)