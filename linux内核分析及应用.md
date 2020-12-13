# linux内核分析及应用 读书笔记
## 前序
[书的链接](https://item.jd.com/12403795.html)

## 调试手段
### gdb提供了调试线程的方法(CH.1.5.1)
1. 跟踪子进程
`(gdb)set follow-fork-mode child`
2. 跟踪父进程
`(gdb)set follow-fork-mode parent`
3. 设置gdb在fork时询问跟踪哪一个进程
`(gdb)set follow-fork-mode ask`

### strace工具使用(CH.1.5.3)
1. strace是linux提供的一个工具，常用来跟踪进程执行时的系统调用和所接收的信号
```
#strace cat /dev/null 
execve("/bin/cat", ["cat", "/dev/null"], 0x7ffc15ecee58 /* 28 vars */) = 0
brk(NULL)                               = 0x20b7000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f4b971a1000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=127613, ...}) = 0
mmap(NULL, 127613, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4b97181000
close(3)                                = 0
open("/lib64/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0`&\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2156240, ...}) = 0
mmap(NULL, 3985920, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f4b96bb3000
mprotect(0x7f4b96d76000, 2097152, PROT_NONE) = 0
mmap(0x7f4b96f76000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c3000) = 0x7f4b96f76000
mmap(0x7f4b96f7c000, 16896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f4b96f7c000
close(3)                                = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f4b97180000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f4b9717e000
arch_prctl(ARCH_SET_FS, 0x7f4b9717e740) = 0
mprotect(0x7f4b96f76000, 16384, PROT_READ) = 0
mprotect(0x60b000, 4096, PROT_READ)     = 0
mprotect(0x7f4b971a2000, 4096, PROT_READ) = 0
munmap(0x7f4b97181000, 127613)          = 0
brk(NULL)                               = 0x20b7000
brk(0x20d8000)                          = 0x20d8000
brk(NULL)                               = 0x20d8000
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=106176928, ...}) = 0
mmap(NULL, 106176928, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f4b90670000
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
open("/dev/null", O_RDONLY)             = 3
fstat(3, {st_mode=S_IFCHR|0666, st_rdev=makedev(1, 3), ...}) = 0
fadvise64(3, 0, 0, POSIX_FADV_SEQUENTIAL) = 0
read(3, "", 65536)                      = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```
2. strace 使用-c 参数可以统计每一次系统调用所执行的时间、次数和出错的次数等
```
#strace -c cat /dev/null 
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 24.24    0.000508          63         8           mmap
 13.98    0.000293          48         6           close
 12.40    0.000260          65         4           open
 12.12    0.000254          50         5           fstat
 11.31    0.000237          59         4           mprotect
  6.87    0.000144          36         4           brk
  6.39    0.000134          67         2           read
  4.29    0.000090          90         1         1 access
  3.34    0.000070          70         1           munmap
  2.77    0.000058          58         1           arch_prctl
  2.29    0.000048          48         1           fadvise64
  0.00    0.000000           0         1           execve
------ ----------- ----------- --------- --------- ----------------
100.00    0.002096                    38         1 total
```
### systemTap(CH.1.5.4)
#### 为什么需要用到systemtap
	假如现在有这么一个需求：需要获取正在运行的 Linux 系统的信息，
如我想知道系统什么时候发生系统调用，发生的是什么系统调用等这些信息，有什么解决方案呢？

1. 最原始的方法是，找到内核系统调用的代码，加上我们需要获得信息的代码、重新编译内核、
安装、选择我们新编译的内核重启。这种做法对于内核开发人员简直是梦魇，
因为一遍做下来至少得需要1个多小时，不仅破坏了原有内核代码，而且如果换了一个需求又得重新做一遍上面的工作。
所以，这种调试内核的方法效率是极其底下的。
2. 之后内核引入了一种Kprobe机制，可以用来动态地收集调试和性能信息的工具，
是一种非破坏性的工具，用户可以用它跟踪运行中内核任何函数或执行的指令等。
相比之前的做法已经有了质的提高了，但Kprobe并没有提供一种易用的框架，
用户需要自己去写模块，然后安装，对用户的要求还是蛮高的。
3. systemtap 是利用Kprobe 提供的API来实现动态地监控和跟踪运行中的Linux内核的工具，相比Kprobe，
systemtap更加简单，提供给用户简单的命令行接口，以及编写内核指令的脚本语言。
对于开发人员，systemtap是一款难得的工具。

systemtap 的核心思想是定义一个事件（event），以及给出处理该事件的句柄（Handler）。
当一个特定的事件发生时，内核运行该处理句柄，就像快速调用一个子函数一样，
处理完之后恢复到内核原始状态。这里有两个概念：

事件（Event）：systemtap 定义了很多种事件，例如进入或退出某个内核函数、定时器时间到、整个systemtap会话启动或退出等等。
句柄（Handler）：就是一些脚本语句，描述了当事件发生时要完成的工作，通常是从事件的上下文提取数据，将它们存入内部变量中，或者打印出来。
Systemtap 工作原理是通过将脚本语句翻译成C语句，编译成内核模块。模块加载之后，将所有探测的事件以钩子的方式挂到内核上，当任何处理器上的某个事件发生时，相应钩子上句柄就会被执行。最后，当systemtap会话结束之后，钩子从内核上取下，移除模块。整个过程用一个命令 stap 就可以完成。

下面将会介绍systemtap的安装、systemtap的工作原理以及几个简单的示例
#### systemtap的安装
##### 方法一 手动编译安装内核
1. 编译内核需要加上 -g标识和以下内核配置
```
CONFIG_DEBUG_INFO
CONFIG_KPROBES
CONFIG_DEBUG_FS
CONFIG_RELAY
```	
2. 编译systemtap的源码
```	
git clone git://sources.redhat.com/git/systemtap.git
./configure --with-elfutils=~/Document/elfutils-0.156
make 
make install
```	
##### 方法二 安装发行版本linux的可调试内核镜像进行安装
apt-get在线安装systemtap `sudo apt-get install systemtap`

#### systemtap 测试示例
1. 打印hello systemtap
以root用户或者具有sudo权限的用户运行以下命令：
`$stap -ve 'probe begin { log("hello systemtap!") exit() }'`

如果安装正确，会得到如下类似的输出结果
```
Pass 1: parsed user script and 96 library script(s) using 55100virt/26224res/2076shr/25172data kb, in 120usr/0sys/119real ms.
Pass 2: analyzed script: 1 probe(s), 2 function(s), 0 embed(s), 0 global(s) using 55496virt/27016res/2172shr/25568data kb, in 0usr/0sys/4real ms.
Pass 3: translated to C into "/tmp/stapYqNuF9/stap_e2d1c1c9962c809ee9477018c642b661_939_src.c" using 55624virt/27380res/2488shr/25696data kb, in 0usr/0sys/0real ms.
Pass 4: compiled C into "stap_e2d1c1c9962c809ee9477018c642b661_939.ko" in 1230usr/160sys/1600real ms.
Pass 5: starting run.
hello systemtap!
Pass 5: run completed in 0usr/10sys/332real ms.
```
2. 打印4s内所有open系统调用的信息
创建systemtap脚本文件test2.stp:
```
#!/usr/bin/stap

probe begin 
{
    log("begin to probe")
}

probe syscall.open
{
    printf ("%s(%d) open (%s)\n", execname(), pid(), argstr)
}

probe timer.ms(4000) # after 4 seconds
{
    exit ()
}

probe end
{
    log("end to probe")
}
```
将该脚本添加可执行的权限 chmod +x test2.stp ，使用./test2.stp 运行该脚本，即可打印4s内所有open系统调用的信息，打印格式为：进程名（进程号）打开什么文件。 大家可以自行去测试，如果两个示例都能正确运行，基本上算是安装成功了！

### Dtrace工具(CH.1.5.5)
Dtrace是Oracle旗下的一款基于linux的监控程序，它可以基于D语言编写脚本来实现你想要的监控功能。

## 系统调用(CH.4.4)
系统调用syscall是linux系统中非常重要的概念，操作系统的核心功能，如进程、内存、IO、文件系统等，
都是以系统调用的方式提供给用户态程序的。

## 时钟中断(CH.4.5)
时钟中断对操作系统来讲是个很重要的概念，特别是像linux这样的分时操作系统，CPU不能被某几个进程
独占，所以需要通过对时间片的划分进行切换。所以时钟中断是进程实现被动切换的机制。
要产生时钟中断，必须通过硬件来完成，一般情况下，CPU都会通过中断控制器连接8259A这样的芯片，
来产生频率恒定的时钟中断。

## 信号处理机制(CH.4.6)
在linux中，用于通知进程触发响应的事件，这就是信号(signal)处理机制。
linux把它包装为系统调用给用户态程序进行使用，信号回调注册可以通过以下
两个系统调用来完成。
1. sigaction系统调用
2. signal系统调用






