---
title: ceph之docker内crimson编译
subtitle: tech
date: 2021-06-23 09:21:25
updated: 
tags: [ceph, docker, 编译]
categories: [专业]
---

# 配置

配置了一个`CentOS 8`的容器, 执行下以下内容

``` bash
WITH_SEASTAR=true ./install-deps.sh
mkdir build && cd build
# 以下这步官网里没提, 导致失败的.
# . /opt/rh/gcc-toolset-9/enable
cmake -DWITH_SEASTAR=ON -DWITH_MGR_DASHBOARD_FRONTEND=OFF ..
make -j18 crimson-osd
```

用服务器跑很快就失败了...发现主要是少了这个一点`. /opt/rh/gcc-toolset-9/enable`, 导致默认用的是`gcc 8`


好像找到个`Dockerfile.in`, 这里好像匹配了`jenkins`里面的操作, `do_cmake.sh -DWITH_SEASTAR=ON`


归档一下`docker save sean10/centos_ceph_clion:pacific_v1.0.0 | gzip > centos_ceph_clion_pacific_v1.0.0.tar.gz `

提交到`[sean10/centos\_ceph\_clion Tags](https://hub.docker.com/repository/docker/sean10/centos_ceph_clion/tags?page=1&ordering=last_updated)`这里了.

# docker压缩优化

``` bash
docker history sean10/centos_ceph_clion:pacific_v1.1.1
```
看了下, 后面我所有ssh上去做的复制操作都没有, 我的代码因为需要clion上做同步然后Cmake, 直接把代码放进去并不能省去上传的操作, image容量在clion重新上传一次之后还是被会拉大, 因此暂时不考虑压缩了.


## 报错
### CMAKE_CXX_COMPILER could be found.
我看了下`gcc`已经装了8.4版本, 那就是`cmake`没能识别到gcc的环境变量?

嗯, 我没装`gcc-c++`, 所以的确识别不到.

### target "seastar" links to target "lz4::lz4" but the target was not found.
好像缺失的`lz4`的库?

姑且试了下, 好像是`/usr/lib64/pkgconfig/lz4.pc`不存在,但是实际上`/usr/lib64/pkgconfig/liblz4.pc`存在, 我做了个软链, 过了第一个, 但是后面还是报了上面这条报错.

--debug-output

CMake Warning (dev) at /usr/share/cmake/Modules/FindPackageHandleStandardArgs.cmake:273 (message):
  The package name passed to `find_package_handle_standard_args` (LZ4) does               
  not match the name of the calling package (lz4).  This can lead to problems             
  in calling code that expects `find_package` result variables (e.g.,                     
  `_FOUND`) to follow a certain pattern.                                                  
Call Stack (most recent call first): 
  cmake/modules/Findlz4.cmake:30 (find_package_handle_standard_args)    
  src/CMakeLists.txt:335 (_find_package)
  src/seastar/cmake/SeastarDependencies.cmake:98 (find_package)                           
  src/seastar/CMakeLists.txt:330 (seastar_find_dependencies)                              
This warning is for project developers.  Use -Wno-dev to suppress it. 

### Could NOT find LZ4 (missing: LZ4_LIBRARY) 
好像是`FindPackageHandleStandardArgs`这个函数是类似`find_package`的.

好像是我理解错了, 这个就是单纯在找`lz4-devel`这个库提供的内容. 而现在我卸载掉这个`devel`库, 的确还是报相同的错误. 那就是这个库提供的内容没有在他们的搜索路径里导致的

好像`FindLZ4.cmake`和`Findlz4.cmake`本身是不一样的. 我试验了一下, 直接把这几个文件作为`CMakeLists.txt`的时候, 后者那个小写文件是能够找到的, 而第一个则是找不到的...

但是我把第二个文件覆盖掉第一个问题, 则是不工作的. 

``` log
-- Checking for one of the modules 'liblz4'                                                       
CMake Warning (dev) at /usr/share/cmake/Modules/FindPackageHandleStandardArgs.cmake:273 (message):
  The package name passed to `find_package_handle_standard_args` (lz4) does                       
  not match the name of the calling package (LZ4).  This can lead to problems                     
  in calling code that expects `find_package` result variables (e.g.,                             
  `_FOUND`) to follow a certain pattern.                                                          
Call Stack (most recent call first):                                                              
  cmake/modules/FindLZ4.cmake:45 (find_package_handle_standard_args)                              
  CMakeLists.txt:325 (find_package)                                                               
This warning is for project developers.  Use -Wno-dev to suppress it.                             
                                                                                                  
-- Found lz4: /usr/lib64/liblz4.so (found version "1.8.3")                                        
-- Looking for LZ4_compress_default                                                    
-- Looking for LZ4_compress_default - found                                                       
```
所以是不是有地方定义了大写的`LZ4:LZ4`, 所以才导致必须有一个大写的导入.

奇怪, 为什么我的代码里这个目录不存在一个几乎同名的`findlz4.cmake`, 但是在设备上,却在`findLZ4.cmake`同目录里有一个`findlz4.cmake`呢?

这个目录疑似是生成的, 因为我改了以后, 这里的findLZ4.cmake就不存在了.

了解一下cmake的工作机制.

但是按照Cmake的工作机制, 理论上这个位置就不是一个动态生成的路径啊?

这里一直提到的`The package name passed to find_package_handle_standard_args (lz4) does not match the name of the calling package (LZ4).  This can lead to problems in calling code that expects `find_package` result variables (e.g.,            `这里的`calling package`到底是什么意思呢?

根据`cmake`里这段代码

``` cmake
  if (DEFINED CMAKE_FIND_PACKAGE_NAME
      AND NOT FPHSA_NAME_MISMATCHED
      AND NOT _NAME STREQUAL CMAKE_FIND_PACKAGE_NAME)
    message(AUTHOR_WARNING
      "The package name passed to `find_package_handle_standard_args` "
      "(${_NAME}) does not match the name of the calling package "
      "(${CMAKE_FIND_PACKAGE_NAME}). This can lead to problems in clling "
      "code that expects `find_package` result variables (e.g., `_FOUND`) "
      "to follow a certain pattern.")
  endif ()

``CMAKE_FIND_PACKAGE_NAME``
  the ``<PackageName>`` which is searched for
```
也就是调用`find_package(LZ4)`了.

```
  src/seastar/cmake/SeastarDependencies.cmake:98 (find_package
  src/seastar/CMakeLists.txt:330 (seastar_find_dependencies)  
``` 

所以是来自这个文件, 传递了`lz4`为要找的包, 但是实际上只找到了`findLZ4.cmake`. 但是他隶属于`seastar`目录, 在那个目录里, 有一个findlz4小写的. 所以我怀疑是嵌套的`cmake`作用域之类的问题?

build目录下有一个`Makefile.cmake`如何理解?

这个文件是从`CMakeLists.txt`里生成出来的? 的确是生辰出来的...

Boost库还没下载下来, 所以直接编译`seastar`是肯定成功不了的.

最后复制了seastar目录下的`cmake/Findlz4.cmake`到了`cmake/modules`目录下, 和`FindLZ4.cmake`在一起, 就能过了.

不过我推测,应该还有个`CMAKE_FIND_ROOT_PATH`这种东西可能在工作, 可以让`seastar`目录在加载时, 自动使用当前目录的`cmake`来进行`find_package`操作, 不过目前没找到...

### cannot deduce template arguments for 'tuple'  
这次是编译报错. 看着比较像是c++支持语言版本不足, 看了下, 官方的`jenkins`里是有安装了`gcc-toolset-9`这个操作的, 所以按理来说应该是用`gcc 9`版本编译才对.

果然, 在`ceph.spec.in`里是有加载这个`toolset`的操作的.
```
. /opt/rh/gcc-toolset-9/enable
```

这个`ceph.spec.in`是怎么生成的暂时没找到, 不过有个`make-dist`的脚本,这里有生成的操作,  这里触发应该有用吧

通过`make -n`可以看到使用的`c++`的路径不对, 是低版本的, 果然, 我用的`Makefile`版本不对. 必须得更新一下. 删掉整个`build`目录, 先执行`. /opt/rh/gcc-toolset-9/enable`, 这次生成的`Makefile`版本对了.

# Reference
1. [Crimson developer documentation — Ceph Documentation](https://docs.ceph.com/en/latest/dev/crimson/index.html)
2. [crimson — Ceph Documentation](https://docs.ceph.com/en/latest/dev/crimson/crimson/)