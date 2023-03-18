对于没有 debug 信息的代码，step into 会调到 Disassembly 界面。

平时要看汇编码的话 LLDB 里用 `disas` 命令，还是挺方便的（虽然没 VS 方便）

看到上面这句话，那我算是明白了，只是单纯因为没有debug info吧，虽然直接在设备上debug的时候是显示有的

实际上找到了，@exe这类后缀的目录里的文件就是符号表的调试文件，但是gdbserver的时候并不加载，导致很麻烦的一点，在clion里的gdb是运行在本机的，他没有那部分符号表。


set sysroot 与 set solib-absolute-prefix 是同一条命令，实际上，set sysroot是set solib-absolute-prefix 的别名。

　　2. set solib-search-path设置动态库的搜索路径，该命令可设置多个搜索路径，路径之间使用“:”隔开（在linux中为冒号，DOS和Win32中为分号）。

　　3. set solib-absolute-prefix 与 set solib-search-path 的区别：

　　总体上来说solib-absolute-prefix设置库的绝对路径前缀，只对绝对路径有效；而solib-search-path设置库的搜索路径，对绝对路径和相对路径均起作用。

进入gdb

target remote 192.168.123.107:123




define target hookpost-remote
set solib-search-path /path/to/my.so:/path/to/sysroot:/path/to/vendorlibs
break main  # if you also need the debug sessions to pause at the beginning
end

gdb调试远程环境，提示File format not recognized

emm, 居然是mac上的gdb bug，唉，重新安装了下最新的就好了

### 远程多线程调试

$ gdbserver --multi localhost:2000
Listening on port 2000
On Host,

$ gdb

(gdb) target extended-remote 192.168.1.10:2000
Remote debugging using 192.168.1.10:2000

(gdb) (gdb) set remote exec-file /my_prg
(gdb) file /my_prg 
Reading symbols from /my_prg...(no debugging symbols found)...done.
(gdb) b main
Note: breakpoint 1 also set at pc 0x400550.
Breakpoint 2 at 0x400550
(gdb) run
Starting program: /my_prg
Breakpoint 1, 0x0000000000400550 in main ()


# 反编译
从debuginfo里目前初步来看, 得到的结果预期也看不到啥理想的内容. 暂时放弃

[java \- Decompile C code with debug info? \- Stack Overflow](https://stackoverflow.com/questions/15609440/decompile-c-code-with-debug-info)


info r 打印寄存器内


# addr2line
调试segfault问题

将ip后面地址减去 行最后的 库后面的地址 取得地址
``` bash
addr2line -e xxx.so -f 0x26ff04
```

一些高级技巧

[gdb 基础功能使用 \| Runsisi's Blog](https://runsisi.com/2019/01/21/gdb-basic/)



# 热补丁


目前通过`gdb -p $pid --batch -x xxx.patch --symbol xxx.debug`前台是可以正常运行的, 但是直接推后台的话, 就会无法进入断点. 

通过尝试Ctrl+z, 手动推后台, 发现不进断点部分没问题, 但是如果走到我的断点部分也会卡住. 直到我把进程重新呼叫回前台, 才能进入断点.

这个是不是就跟gdb进断点的机制有点关系了? 所以是不是跟SIGTRAP信号有关系?


* SIGTRAP
* SIGINT 可以让他继续?

gdb通过给进程发送SIGSTOP让他变成自己的子进程

[c\+\+ \- gdb stops when using watchpoints \(as if ctrl\-Z\) \- Stack Overflow](https://stackoverflow.com/questions/64980070/gdb-stops-when-using-watchpoints-as-if-ctrl-z)

> unix环境下，当一个进程以后台形式启动，但尝试去读写控制台终端时，将会触发SIGTTIN（读）和SIGTTOU（写）信号量，接着，进程将会暂停（linux默认情况下），read/write将会返回错误。这个时候，shell将会发送通知给用户，提醒用户切换此进程为前台进程，以便继续执行。由后台切换至前台的方式是fg命令，前台转为后台则为CTRL+Z快捷键。
> 
> 那么问题来了，如何才能在不把进程切换至前台的情况下，读写控制器不会被暂停？答案：只要忽略SIGTTIN和SIGTTOU信号量即可：signal(SIGTTOU, SIG_IGN)。
> 
> stty stop/-stop命令是用于设置收到SIGTTOU信号量后是否执行暂停，因为有些系统的默认行为不一致，比如mac是默认忽略，而linux是默认启用。stty -a可以查看当前tty的配置参数
> 
> [后台进程读/写控制台触发SIGTTIN/SIGTTOU信号量 \- 简书](https://www.jianshu.com/p/1449c0abcf1e)


trap可以忽略信号欸

> `trap 'echo trap handling...;rm -rf /tmp/$BASHPID$BASHPID;echo TEMP file cleaned;exit' SIGINT SIGTERM SIGQUIT SIGHUP`


[为shell布置陷阱：trap捕捉信号方法论 \- 骏马金龙 \- 博客园](https://www.cnblogs.com/f-ck-need-u/p/7454174.html)

根据这篇讲的, [Alan Hayward \- \[PATCH\] Supress SIGTTOU when handling errors](https://sourceware.org/legacy-ml/gdb-patches/2019-05/msg00361.html)

确实很像, 说到这个就意识到了, 后台进程对SIGTTOU的处置确实是不太一样的.

确实, 通过更换成gdb-10, 确实解决了这个问题.

但是如果在当前窗口开tailf, 可能会把输出的信号给拦截掉了

yum install --downloadonly --downloaddir=./ devtoolset-11-gdb


TODO: emm, 根据磁盘压力来说, 是判断不出哪个osd给他发的数据的.

其他的好像是从start_recovery_op开始的.





unix环境下，当一个进程以后台形式启动，但尝试去读写控制台终端时，将会触发SIGTTIN（读）和SIGTTOU（写）信号量，接着，进程将会暂停（linux默认情况下），read/write将会返回错误。这个时候，shell将会发送通知给用户，提醒用户切换此进程为前台进程，以便继续执行。由后台切换至前台的方式是fg命令，前台转为后台则为CTRL+Z快捷键。

那么问题来了，如何才能在不把进程切换至前台的情况下，读写控制器不会被暂停？答案：只要忽略SIGTTIN和SIGTTOU信号量即可：signal(SIGTTOU, SIG_IGN)。

stty stop/-stop命令是用于设置收到SIGTTOU信号量后是否执行暂停，因为有些系统的默认行为不一致，比如mac是默认忽略，而linux是默认启用。stty -a可以查看当前tty的配置参数。

作者：丶Em1tu0F
链接：https://www.jianshu.com/p/1449c0abcf1e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# 概念

用readelf -S分别查看strip前后两个so，会发现strip后的so只少了symtab和strtab两个section，这两个section保存的符号主要用于调试，而保存链接符号的dynsym和dynstr都没有被strip掉。

`<linkers and loaders>`


依赖proc, 获取maps

> 在这个目录中，有两个伪文件，分别是maps文件和mem文件。

> maps文件包含分配给二进制文件的所有内存区域架构，以及所有包含的动态库。不过，这个信息现在相对敏感，因为每个库位置的偏移量是由ASLR随机化的。

> mem文件提供了流程使用的完整内存空间的稀疏架构，结合从maps文件获得的偏移量，可以使用mem文件读取和写入进程的内存空间。如果偏移量是错误的，或者从开始位置按顺序读取文件，将返回读写错误，因为这相当于是读取不可访问的未分配内存。





[无需Ptrace就能实现Linux进程间代码注入 \- 知乎](https://zhuanlan.zhihu.com/p/29264608)

[ConFoo 2019: Profiling and tracing with ptrace, perf and SystemTap \- Speaker Deck](https://speakerdeck.com/tiran/confoo-2019-profiling-and-tracing-with-ptrace-perf-and-systemtap?slide=2)

### 可能有关, symbol覆盖二进制

1，通过gdb启动参数-s指定：
1
2
3
[root@lenky ~]# gdb -s /home/work/gdb/.debug/t.debug -e /tmp/t -q
Reading symbols from /home/work/gdb/.debug/t.debug...done.
(gdb)
注意：可执行程序必须通过-e指定，否则貌似gdb会拿它覆盖-s参数，比如如下：




### 符号表 弱引用

> 强引用的意思是在链接的阶段，符号要有正确的寻址，若没找到该符号的定义，连接器就会报未定义的错误。弱引用的意思是在连接阶段，若能找到该符号的定义，那么连接器将引用该符号；若没被定义也没关系，连接器不报错，而是默认其为零。我们可以使用gcc的参数__attribute__ ((weakref))来将一个强引用转换成弱引用。

> 在Linux下，我们可以通过版本脚本version-script来控制符号版本。只需要在编译动态库是加上-Wl,--version-script xxx即可
> 

[符号表的一些事一些情 \- HHP的博客 \| HHP Blog](https://huhaipeng.top/2019/09/15/%E7%AC%A6%E5%8F%B7%E8%A1%A8%E7%9A%84%E4%B8%80%E4%BA%9B%E4%BA%8B%E4%B8%80%E4%BA%9B%E6%83%85/)


# std调试


[STLSupport \- GDB Wiki](https://sourceware.org/gdb/wiki/STLSupport)


从这里下载stl-views.gdb然后重命名成~/.gdbinit, 然后正常gdb的时候pset, pvector即可

