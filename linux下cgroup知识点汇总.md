# 简介
	cgroups(Control Groups) 是 linux 内核提供的一种机制，
这种机制可以根据需求把一系列系统任务及其子任务整合(或分隔)到按资源划分等级的不同组内，
从而为系统资源管理提供一个统一的框架。
简单说，cgroups 可以限制、记录任务组所使用的物理资源。
本质上来说，cgroups 是内核附加在程序上的一系列钩子(hook)，通过程序运行时对资源的调度触发相应的钩子
以达到资源追踪和限制的目的。

# 为什么要了解 cgroups
	我们可以通过linux的cgroup，来限制控制进程的资源使用，比如限制进程对cpu的使用、内存的使用等等。
以此来合理分配控制应用进程对系统资源的使用。

# cgroups 的主要作用
实现 cgroups 的主要目的是为不同用户层面的资源管理提供一个统一化的接口。从单个任务的资源控制到操作系统层面的虚拟化，cgroups 提供了四大功能：

1. 资源限制：cgroups 可以对任务是要的资源总额进行限制。比如设定任务运行时使用的内存上限，一旦超出就发 OOM。
2. 优先级分配：通过分配的 CPU 时间片数量和磁盘 IO 带宽，实际上就等同于控制了任务运行的优先级。
3. 资源统计：cgoups 可以统计系统的资源使用量，比如 CPU 使用时长、内存用量等。这个功能非常适合当前云端产品按使用量计费的方式。
4. 任务控制：cgroups 可以对任务执行挂起、恢复等操作。

# 相关概念
Task(任务) 在 linux 系统中，内核本身的调度和管理并不对进程和线程进行区分，只是根据 clone 时传入的参数的不同来从概念上区分进程和线程。这里使用 task 来表示系统的一个进程或线程。

Cgroup(控制组) cgroups 中的资源控制以 cgroup 为单位实现。Cgroup 表示按某种资源控制标准划分而成的任务组，包含一个或多个子系统。一个任务可以加入某个 cgroup，也可以从某个 cgroup 迁移到另一个 cgroup。

Subsystem(子系统) cgroups 中的子系统就是一个资源调度控制器(又叫 controllers)。比如 CPU 子系统可以控制 CPU 的时间分配，内存子系统可以限制内存的使用量。以笔者使用的 Ubuntu 16.04.3 为例，其内核版本为 4.10.0，支持的 subsystem 如下( cat /proc/cgroups)：
	blkio         对块设备的 IO 进行限制。
	cpu           限制 CPU 时间片的分配，与 cpuacct 挂载在同一目录。
	cpuacct     生成 cgroup 中的任务占用 CPU 资源的报告，与 cpu 挂载在同一目录。
	cpuset       给 cgroup 中的任务分配独立的 CPU(多处理器系统) 和内存节点。
	devices     允许或禁止 cgroup 中的任务访问设备。
	freezer      暂停/恢复 cgroup 中的任务。
	hugetlb     限制使用的内存页数量。              
	memory      对 cgroup 中的任务的可用内存进行限制，并自动生成资源占用报告。
	net_cls      使用等级识别符（classid）标记网络数据包，
				这让 Linux 流量控制器（tc 指令）可以识别来自特定 
				cgroup 任务的数据包，并进行网络限制。
	net_prio    允许基于 cgroup 设置网络流量(netowork traffic)的优先级。
	perf_event  允许使用 perf 工具来监控 cgroup。
	pids          限制任务的数量。

Hierarchy(层级) 层级有一系列 cgroup 以一个树状结构排列而成，每个层级通过绑定对应的子系统进行资源控制。
层级中的 cgroup 节点可以包含零个或多个子节点，子节点继承父节点挂载的子系统。
一个操作系统中可以有多个层级。

# cgroups 的文件系统接口
cgroups 以文件的方式提供应用接口，我们可以通过 mount 命令来查看 cgroups 默认的挂载点：
```
[root@sfos-x86_64 ~]# mount |grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
```

第一行的 tmpfs 说明 /sys/fs/cgroup 目录下的文件都是存在于内存中的临时文件。
第二行的挂载点 /sys/fs/cgroup/systemd 用于 systemd 系统对 cgroups 的支持。
其余的挂载点则是内核支持的各个子系统的根级层级结构。

需要注意的是，在使用 systemd 系统的操作系统中，/sys/fs/cgroup 目录都是由 systemd 在系统启动的过程中挂载的，并且挂载为只读的类型。换句话说，系统是不建议我们在 /sys/fs/cgroup 目录下创建新的目录并挂载其它子系统的。

# cgroup对应子系统介绍
/sys/fs/cgroup 目录下是各个子系统的根目录
```
[root@sfos-x86_64 ~]# ll /sys/fs/cgroup/
total 0
dr-xr-xr-x 2 root root  0 Apr 11 00:29 blkio
lrwxrwxrwx 1 root root 11 Apr 11 00:29 cpu -> cpu,cpuacct
lrwxrwxrwx 1 root root 11 Apr 11 00:29 cpuacct -> cpu,cpuacct
dr-xr-xr-x 2 root root  0 Apr 11 00:29 cpu,cpuacct
dr-xr-xr-x 2 root root  0 Apr 11 00:29 cpuset
dr-xr-xr-x 5 root root  0 Apr 11 00:29 devices
dr-xr-xr-x 2 root root  0 Apr 11 00:29 freezer
dr-xr-xr-x 2 root root  0 Apr 11 00:29 hugetlb
dr-xr-xr-x 5 root root  0 Apr 11 00:29 memory
lrwxrwxrwx 1 root root 16 Apr 11 00:29 net_cls -> net_cls,net_prio
dr-xr-xr-x 2 root root  0 Apr 11 00:29 net_cls,net_prio
lrwxrwxrwx 1 root root 16 Apr 11 00:29 net_prio -> net_cls,net_prio
dr-xr-xr-x 2 root root  0 Apr 11 00:29 perf_event
dr-xr-xr-x 5 root root  0 Apr 11 00:29 pids
dr-xr-xr-x 2 root root  0 Apr 11 00:29 rdma
dr-xr-xr-x 6 root root  0 Apr 11 00:29 systemd
```

## 子系统之 memory
```
[root@sfos-x86_64 ~]# ll /sys/fs/cgroup/memory/
total 0
-rw-r--r--  1 root root 0 Apr 12 17:00 cgroup.clone_children
--w--w--w-  1 root root 0 Apr 12 17:00 cgroup.event_control
-rw-r--r--  1 root root 0 Apr 12 17:00 cgroup.procs
-r--r--r--  1 root root 0 Apr 12 17:00 cgroup.sane_behavior
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.failcnt
--w-------  1 root root 0 Apr 12 17:00 memory.force_empty
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.kmem.failcnt
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.kmem.limit_in_bytes
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.kmem.max_usage_in_bytes
-r--r--r--  1 root root 0 Apr 12 17:00 memory.kmem.slabinfo
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.kmem.tcp.failcnt
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.kmem.tcp.limit_in_bytes
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.kmem.tcp.max_usage_in_bytes
-r--r--r--  1 root root 0 Apr 12 17:00 memory.kmem.tcp.usage_in_bytes
-r--r--r--  1 root root 0 Apr 12 17:00 memory.kmem.usage_in_bytes
-rw-r--r--  1 root root 0 Apr 11 00:30 memory.limit_in_bytes
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.max_usage_in_bytes
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.memsw.failcnt
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.memsw.limit_in_bytes
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.memsw.max_usage_in_bytes
-r--r--r--  1 root root 0 Apr 12 17:00 memory.memsw.usage_in_bytes
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.move_charge_at_immigrate
-r--r--r--  1 root root 0 Apr 12 17:00 memory.numa_stat
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.oom_control
----------  1 root root 0 Apr 12 17:00 memory.pressure_level
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.soft_limit_in_bytes
-r--r--r--  1 root root 0 Apr 12 17:00 memory.stat
-rw-r--r--  1 root root 0 Apr 12 17:00 memory.swappiness
-r--r--r--  1 root root 0 Apr 12 17:00 memory.usage_in_bytes
-rw-r--r--  1 root root 0 Apr 11 00:30 memory.use_hierarchy
-rw-r--r--  1 root root 0 Apr 12 17:00 notify_on_release
-rw-r--r--  1 root root 0 Apr 12 17:00 release_agent
drwxr-xr-x  7 root root 0 Apr 11 00:29 super.slice
drwxr-xr-x 54 root root 0 Apr 11 00:29 system.slice
-rw-r--r--  1 root root 0 Apr 12 17:00 tasks
drwxr-xr-x  3 root root 0 Apr 11 00:30 user.slice
```
这些文件就是 cgroups 的 memory 子系统中的根级设置。
比如 memory.limit_in_bytes 中的数字用来限制进程的最大可用内存，
memory.swappiness 中保存着使用 swap 的权重等等。

## 子系统之 cpu
```
[root@sfos-x86_64 ~]# ll /sys/fs/cgroup/cpu/
total 0
-rw-r--r-- 1 root root 0 Apr 12 17:01 cgroup.clone_children
-rw-r--r-- 1 root root 0 Apr 11 00:29 cgroup.procs
-r--r--r-- 1 root root 0 Apr 12 17:01 cgroup.sane_behavior
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.stat
-rw-r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage_all
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage_percpu
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage_percpu_sys
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage_percpu_user
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage_sys
-r--r--r-- 1 root root 0 Apr 12 17:01 cpuacct.usage_user
-rw-r--r-- 1 root root 0 Apr 12 17:01 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Apr 12 17:01 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Apr 12 17:01 cpu.rt_period_us
-rw-r--r-- 1 root root 0 Apr 12 17:01 cpu.rt_runtime_us
-rw-r--r-- 1 root root 0 Apr 12 17:01 cpu.shares
-r--r--r-- 1 root root 0 Apr 12 17:01 cpu.stat
-rw-r--r-- 1 root root 0 Apr 12 17:01 notify_on_release
-rw-r--r-- 1 root root 0 Apr 12 17:01 release_agent
-rw-r--r-- 1 root root 0 Apr 12 17:01 tasks
```
这些文件就是 cgroups 的 cpu 子系统中的根级设置。
cpuacct.xxx 统计cgroup的CPU的使用率
cpu.xxx 用来限制cgroup的CPU使用率

### cpu.cfs_period_us & cpu.cfs_quota_us
cfs_period_us用来配置时间周期长度，
cfs_quota_us用来配置当前cgroup在设置的周期长度内所能使用的CPU时间数，
两个文件配合起来设置CPU的使用上限。两个文件的单位都是微秒（us），
cfs_period_us的取值范围为1毫秒（ms）到1秒（s），cfs_quota_us的取值大于1ms即可，
如果cfs_quota_us的值为-1（默认值），表示不受cpu时间的限制
```
1.限制只能使用1个CPU（每250ms能使用250ms的CPU时间）
    # echo 250000 > cpu.cfs_quota_us /* quota = 250ms */
    # echo 250000 > cpu.cfs_period_us /* period = 250ms */

2.限制使用2个CPU（内核）（每500ms能使用1000ms的CPU时间，即使用两个内核）
    # echo 1000000 > cpu.cfs_quota_us /* quota = 1000ms */
    # echo 500000 > cpu.cfs_period_us /* period = 500ms */

3.限制使用1个CPU的20%（每50ms能使用10ms的CPU时间，即使用一个CPU核心的20%）
    # echo 10000 > cpu.cfs_quota_us /* quota = 10ms */
    # echo 50000 > cpu.cfs_period_us /* period = 50ms */
```

### cpu.shares
shares用来设置CPU的相对值，并且是针对所有的CPU（内核），默认值是1024，
假如系统中有两个cgroup，分别是A和B，A的shares值是1024，B的shares值是512，
那么A将获得1024/(1204+512)=66%的CPU资源，而B将获得33%的CPU资源。
shares有两个特点：
1. 如果A不忙，没有使用到66%的CPU时间，那么剩余的CPU时间将会被系统分配给B，即B的CPU使用率可以超过33%
2. 如果添加了一个新的cgroup C，且它的shares值是1024，那么A的限额变成了1024/(1204+512+1024)=40%，B的变成了20%

从上面两个特点可以看出：
在闲的时候，shares基本上不起作用，只有在CPU忙的时候起作用，这是一个优点。
由于shares是一个绝对值，需要和其它cgroup的值进行比较才能得到自己的相对限额，
而在一个部署很多容器的机器上，cgroup的数量是变化的，所以这个限额也是变化的，
自己设置了一个高的值，但别人可能设置了一个更高的值，所以这个功能没法精确的控制CPU使用率。

### cpu.stat
包含了下面三项统计结果
1. nr_periods : 表示过去了多少个cpu.cfs_period_us里面配置的时间周期
2. nr_throttled : 在上面的这些周期中，有多少次是受到了限制（即cgroup中的进程在指定的时间周期中用光了它的配额）
3. throttled_time : cgroup中的进程被限制使用CPU持续了多长时间(纳秒)

### 实例1
这里以cfs_period_us & cfs_quota_us为例，演示一下如何控制CPU的使用率。
```
#继续使用上面创建的子cgroup： test
#设置只能使用1个cpu的20%的时间
dev@ubuntu:/sys/fs/cgroup/cpu,cpuacct/test$ sudo sh -c "echo 50000 > cpu.cfs_period_us"
dev@ubuntu:/sys/fs/cgroup/cpu,cpuacct/test$ sudo sh -c "echo 10000 > cpu.cfs_quota_us"

#将当前bash加入到该cgroup
dev@ubuntu:/sys/fs/cgroup/cpu,cpuacct/test$ echo $$
5456
dev@ubuntu:/sys/fs/cgroup/cpu,cpuacct/test$ sudo sh -c "echo 5456 > cgroup.procs"

#在bash中启动一个死循环来消耗cpu，正常情况下应该使用100%的cpu（即消耗一个内核）
dev@ubuntu:/sys/fs/cgroup/cpu,cpuacct/test$ while :; do echo test > /dev/null; done

#--------------------------重新打开一个shell窗口----------------------
#通过top命令可以看到5456的CPU使用率为20%左右，说明被限制住了
#不过这时系统的%us+%sy在10%左右，那是因为我测试的机器上cpu是双核的，
#所以系统整体的cpu使用率为10%左右
dev@ubuntu:~$ top
Tasks: 139 total,   2 running, 137 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.6 us,  6.2 sy,  0.0 ni, 88.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   499984 total,    15472 free,    81488 used,   403024 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   383332 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 5456 dev       20   0   22640   5472   3524 R  20.3  1.1   0:04.62 bash

#这时可以看到被限制的统计结果
dev@ubuntu:~$ cat /sys/fs/cgroup/cpu,cpuacct/test/cpu.stat
nr_periods 1436
nr_throttled 1304
throttled_time 51542291833
```

### 实例2 使用cgexec来把进程绑定到指定的cgroup
1. 安装cgexec工具
`dnf install -y libcgroup-tools` or `apt install -y cgroup-bin`

2. 创建自己的cgroup
在我们使用 cgroups 时，最好不要直接在各个子系统的根目录下直接修改其配置文件。推荐的方式是为不同的需求在子系统树中定义不同的节点。比如我们可以在 /sys/fs/cgroup/cpu 目录下新建一个名称为 nick_cpu 的目录：

```
$ cd /sys/fs/cgroup/cpu
$ sudo mkdir ych_cpu
[root@dynamic-005-005-005-002 ych_cpu]# pwd
/sys/fs/cgroup/cpu/ych_cpu
[root@dynamic-005-005-005-002 ych_cpu]# ll 
total 0
-rw-r--r-- 1 root root 0 Apr 10 09:44 cgroup.clone_children
-rw-r--r-- 1 root root 0 Apr 10 09:44 cgroup.procs
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.stat
-rw-r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage_all
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage_percpu
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage_percpu_sys
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage_percpu_user
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage_sys
-r--r--r-- 1 root root 0 Apr 10 09:44 cpuacct.usage_user
-rw-r--r-- 1 root root 0 Apr 10 09:44 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Apr 10 17:27 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Apr 10 09:44 cpu.rt_period_us
-rw-r--r-- 1 root root 0 Apr 10 09:44 cpu.rt_runtime_us
-rw-r--r-- 1 root root 0 Apr 10 09:44 cpu.shares
-r--r--r-- 1 root root 0 Apr 10 09:44 cpu.stat
-rw-r--r-- 1 root root 0 Apr 10 09:44 notify_on_release
-rw-r--r-- 1 root root 0 Apr 10 17:27 tasks
```
让我们通过下面的设置把 CPU 周期限制为总量的十分之一：

```
$ sudo su
$ echo 100000 > ych_cpu/cpu.cfs_period_us
$ echo 10000 > ych_cpu/cpu.cfs_quota_us
```

测试代码:
```
#define _GNU_SOURCE			/* See feature_test_macros(7) */
#include <sched.h>
#include <stdlib.h>
#include <sys/types.h>
#include <signal.h>
#include <unistd.h>
#include <sys/prctl.h>
#include <sys/mman.h>
#include <errno.h>
#include <string.h>

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/resource.h>

#define YCH_LOG(fmt, arg...)                                  \
do                                                            \
{                                                             \
    printf("[%d]ych:%s(%d) "fmt"\r\n", getpid(), __FUNCTION__, __LINE__, ##arg);  \
}                                                             \
while((0))

void bind_cpus(int cpu)
{
	int i;

	cpu_set_t cpu_affinity;

	CPU_ZERO(&cpu_affinity);
	CPU_SET(cpu, &cpu_affinity);

	for (i = 0; i < CPU_SETSIZE; i++) {
		if (CPU_ISSET(i, &cpu_affinity)) {
			YCH_LOG("cpuset_setaffinity(): using cpu #%u", i);
		}
	}
	if (sched_setaffinity(getpid(), sizeof(cpu_set_t), &cpu_affinity) == -1) {
		YCH_LOG("sched_setaffinity() failed");
	}
}

int main(int argc, char **argv)
{
	int index;
	int cpu = 0;
	int pri = 0;
	
	for (index = 0; index < argc; index++) {
		YCH_LOG("argv[%d]:%s", index, argv[index]);
	}

	if (argc < 3) {
		YCH_LOG("input para err:%d", argc);
		return 0;
	}
		
	if (argc >= 2)
		cpu = atoi(argv[1]);
	
	if (argc >= 3)
		pri = atoi(argv[2]);
	
	setpriority(PRIO_PROCESS, 0, pri);
	
	bind_cpus(cpu);
	
	while (1) {
		;
	}
	return 0;
}
```

1. 指定cgroup启动
cgexec -g cpu:ych_cpu ./out 1 -19 &
2. 默认启动
./out 1 -19 &

```
Threads: 540 total,   3 running, 537 sleeping,   0 stopped,   0 zombie
%Cpu(s): 47.3 us,  1.0 sy,  0.0 ni, 49.1 id,  0.0 wa,  1.2 hi,  0.5 si,  1.0 st
MiB Mem :   1829.4 total,    677.5 free,    373.2 used,    778.7 buff/cache
MiB Swap:   2116.0 total,   1847.3 free,    268.7 used.   1204.5 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ P COMMAND
29120 root       1 -19    4328    804    740 R  94.7   0.0   4:44.87 1 out
29171 root       1 -19    4328    692    632 R   0.7   0.0   0:00.02 1 out
```
两个都是死循环的进程，在同一个core，nice都是-19, 一个使用根目录cgroup 优先级远大于 子目录cgroup的程序
```
[root@dynamic-005-005-005-002 ych]# cat /proc/29120/cgroup
12:freezer:/
11:cpuset:/
10:perf_event:/
9:blkio:/user.slice
8:devices:/user.slice
7:hugetlb:/
6:pids:/user.slice/user-1000.slice/session-2.scope
5:rdma:/
4:cpu,cpuacct:/
3:net_cls,net_prio:/
2:memory:/user.slice/user-1000.slice/session-2.scope
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
[root@dynamic-005-005-005-002 ych]# cat /proc/29171/cgroup
12:freezer:/
11:cpuset:/
10:perf_event:/
9:blkio:/user.slice
8:devices:/user.slice
7:hugetlb:/
6:pids:/user.slice/user-1000.slice/session-2.scope
5:rdma:/
4:cpu,cpuacct:/ych_cpu
3:net_cls,net_prio:/
2:memory:/user.slice/user-1000.slice/session-2.scope
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
```

### 注意 
使用cgroup限制CPU的使用率比较纠结，用cfs_period_us & cfs_quota_us吧，
限制死了，没法充分利用空闲的CPU，用shares吧，又没法配置百分比，极其难控制。
总之，使用cgroup的cpu子系统需谨慎

# 相关链接

https://man7.org/linux/man-pages/man7/cgroups.7.html
https://www.kernel.org/doc/Documentation/cgroup-v2.txt
http://www.jinbuguo.com/systemd/systemd.resource-control.html#CPUQuota=
https://www.freedesktop.org/software/systemd/man/systemd.exec.html




















