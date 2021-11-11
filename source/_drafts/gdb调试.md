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