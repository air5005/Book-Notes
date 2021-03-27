[TOC]

# perf 相关资料
https://www.kancloud.cn/wepeng/php/1172704

# perf常用定位性能瓶颈命令

## perf 二级命令
perf --help之后可以看到perf的二级命令

1. archive	解析perf record生成的perf.data文件，显示被注释的代码。
2. annotate 根据数据文件记录的build-id,将所有被采样到的elf文件打包。
			利用此压缩包，可以再任何机器上分析数据文件中记录的采样数据。
3. bench perf中内置的benchmark，目前包括两套针对调度器和内存管理子系统的benchmark。
4. mem	内存存取情况
5. stat	执行某个命令，收集特定进程的性能概况，包括CPI、Cache丢失率等。
6. annotate	解析perf record生成的perf.data文件，显示被注释的代码。
7. buildid-cache 管理perf的buildid缓存，每个elf文件都有一个独一无二的buildid。
			buildid被perf用来关联性能数据与elf文件。
8. buildid-list 列出数据文件中记录的所有buildid。
9. diff 对比两个数据文件的差异。能够给出每个符号（函数）在热点分析上的具体差异。
10. evlist 列出数据文件perf.data中所有性能事件。
11. inject 该工具读取perf record工具记录的事件流，并将其定向到标准输出。
	在被分析代码中的任何一点，都可以向事件流中注入其它事件。
12. kmem 针对内核内存（slab）子系统进行追踪测量的工具
13. kvm 用来追踪测试运行在KVM虚拟机上的Guest OS。
14. list 列出当前系统支持的所有性能事件。包括硬件性能事件、软件性能事件以及检查点。
15. lock 分析内核中的锁信息，包括锁的争用情况，等待延迟等。
16. record 收集采样信息，并将其记录在数据文件中。随后可通过其它工具对数据文件进行分析。
17. report 读取perf record创建的数据文件，并给出热点分析结果。
18. sched 针对调度器子系统的分析工具。
19. script 执行perl或python写的功能扩展脚本、生成脚本框架、读取数据文件中的数据信息等。
20. test perf对当前软硬件平台进行健全性测试，
	可用此工具测试当前的软硬件平台是否能支持perf的所有功能。
21. timechart 针对测试期间系统行为进行可视化的工具
22. top 类似于linux的top命令，对系统性能进行实时分析。
23. trace 关于syscall的工具。
24. probe 用于定义动态检查点。

## perf  top

通过-e指定关注的事件，比如查看造成cache miss最多的函数排行

perf top -e cache-misses

perf top -e task-clock 

perf top -G // 得到调用关系图

perf top-e cache-misses -G // 得到调用关系图

perf top -e cycles // 指定性能事件

perf top -p 23015,32476 //查看这两个进程的cpu cycles使用情况

perf top -s comm,pid,symbol // 显示调用symbol的进程名和进程号

perf top --comms nginx,top // 仅显示属于指定进程的符号

perf top --symbols kfree // 仅显示指定的符号

## perf  list

列出所有能够触发 perf 采样点的事件

Hardware Event 是由 PMU 硬件产生的事件，比如 cache 命中，当您需要了解程序对硬件特性的使用情况时，便需要对这些事件进行采样；

Software Event 是内核软件产生的事件，比如进程切换，tick 数等 ;

Tracepoint event 是内核中的静态 tracepoint 所触发的事件，这些 tracepoint 用来判断程序运行期间内核的行为细节，比如 slab 分配器的分配次数等。

```
List of pre-defined events (to be used in -e):

  alignment-faults                                   [Software event]
  bpf-output                                         [Software event]
  context-switches OR cs                             [Software event]
  cpu-clock                                          [Software event]
  cpu-migrations OR migrations                       [Software event]
  dummy                                              [Software event]
  emulation-faults                                   [Software event]
  major-faults                                       [Software event]
  minor-faults                                       [Software event]
  page-faults OR faults                              [Software event]
  task-clock                                         [Software event]

  duration_time                                      [Tool event]

  msr/tsc/                                           [Kernel PMU event]

  rNNN                                               [Raw hardware event descriptor]
  cpu/t1=v1[,t2=v2,t3 ...]/modifier                  [Raw hardware event descriptor]
   (see 'man perf-list' on how to encode it)

  mem:<addr>[/len][:access]                          [Hardware breakpoint]

  alarmtimer:alarmtimer_cancel                       [Tracepoint event]
  alarmtimer:alarmtimer_fired                        [Tracepoint event]
  alarmtimer:alarmtimer_start                        [Tracepoint event]
  alarmtimer:alarmtimer_suspend                      [Tracepoint event]
  block:block_bio_backmerge                          [Tracepoint event]
```

## perf record

记录信息到perf.data

### 参数详解



### 实际使用命令

1. 记录指定cpu指定时间段内的所有事件

```
perf record  -C 0 -g -- sleep 10 -o perf.data
```

2. 记录指定cpu指定时间段内的所有系统调用 

```
perf record -e syscalls:*  -C 1 -g -- sleep 10 -o perf.data
```

3. 记录指定cpu指定时间段内的指定系统调用 

```
perf record -e raw_syscalls:sys_enter -C 1-8,13-20 -g -- sleep 5
perf record -e raw_syscalls:sys_enter -C 1 -g -- sleep 5
```

## perf report

生成报告

1. 只关注cpu，使用该命令能看到cpu使用率刚好是100%

```
perf report -i perf.data -g --sort=cpu
```

## perf sched

用perf sched 抓取调用信息，可以用于查看调度延迟和唤醒延迟

```
perf sched record -C 1 sleep 10       # 记录指定cpu指定时间段内的调度事件
perf sched latency                    # 查看调度时延
perf sched script                     # 查看调度详情
```

## perf diff

对两个记录进行diff

## perf evlist

列出记录的性能事件

## perf annotate

显示perf.data函数代码

## perf archive

将相关符号打包，方便在其它机器进行分析；

## perf script

将perf.data输出可读性文本

## perf stat
perf stat用于运行指令，并分析其统计结果。虽然perf top也可以指定pid，但是必须先启动应用才能查看信息。

perf stat能完整统计应用整个生命周期的信息。
命令格式为：
```
perf stat [-e <EVENT> | --event=EVENT] [-a] <command>
perf stat [-e <EVENT> | --event=EVENT] [-a] — <command> [<options>]
```

下面简单看一下perf stat 的输出：
```
~/perf$ sudo perf stat
  Performance counter stats for 'system wide':

      40904.820871      cpu-clock (msec)          #    5.000 CPUs utilized          
             18,132      context-switches          #    0.443 K/sec                  
              1,053      cpu-migrations            #    0.026 K/sec                  
              2,420      page-faults               #    0.059 K/sec                  
      3,958,376,712      cycles                    #    0.097 GHz                      (49.99%)
        574,598,403      stalled-cycles-frontend   #   14.52% frontend cycles idle     (49.98%)
      9,392,982,910      stalled-cycles-backend    #  237.29% backend cycles idle      (50.00%)
      1,653,185,883      instructions              #    0.42  insn per cycle         
                                                   #    5.68  stalled cycles per insn  (50.01%)
        237,061,366      branches                  #    5.795 M/sec                    (50.02%)
         18,333,168      branch-misses             #    7.73% of all branches          (50.00%)

       8.181521203 seconds time elapsed
```
cpu-clock：任务真正占用的处理器时间，单位为ms。CPUs utilized = task-clock / time elapsed，CPU的占用率。

context-switches：程序在运行过程中上下文的切换次数。

CPU-migrations：程序在运行过程中发生的处理器迁移次数。Linux为了维持多个处理器的负载均衡，在特定条件下会将某个任务从一个CPU迁移到另一个CPU。

CPU迁移和上下文切换：发生上下文切换不一定会发生CPU迁移，而发生CPU迁移时肯定会发生上下文切换。发生上下文切换有可能只是把上下文从当前CPU中换出，下一次调度器还是将进程安排在这个CPU上执行。

page-faults：缺页异常的次数。当应用程序请求的页面尚未建立、请求的页面不在内存中，或者请求的页面虽然在内存中，但物理地址和虚拟地址的映射关系尚未建立时，都会触发一次缺页异常。另外TLB不命中，页面访问权限不匹配等情况也会触发缺页异常。

cycles：消耗的处理器周期数。如果把被ls使用的cpu cycles看成是一个处理器的，那么它的主频为2.486GHz。可以用cycles / task-clock算出。

stalled-cycles-frontend：指令读取或解码的质量步骤，未能按理想状态发挥并行左右，发生停滞的时钟周期。

stalled-cycles-backend：指令执行步骤，发生停滞的时钟周期。

instructions：执行了多少条指令。IPC为平均每个cpu cycle执行了多少条指令。

branches：遇到的分支指令数。branch-misses是预测错误的分支指令数。

其他常用参数
```
    -a, --all-cpus        显示所有CPU上的统计信息
    -C, --cpu <cpu>       显示指定CPU的统计信息
    -c, --scale           scale/normalize counters
    -D, --delay <n>       ms to wait before starting measurement after program start
    -d, --detailed        detailed run - start a lot of events
    -e, --event <event>   event selector. use 'perf list' to list available events
    -G, --cgroup <name>   monitor event in cgroup name only
    -g, --group           put the counters into a counter group
    -I, --interval-print <n>
                          print counts at regular interval in ms (>= 10)
    -i, --no-inherit      child tasks do not inherit counters
    -n, --null            null run - dont start any counters
    -o, --output <file>   输出统计信息到文件
    -p, --pid <pid>       stat events on existing process id
    -r, --repeat <n>      repeat command and print average + stddev (max: 100, forever: 0)
    -S, --sync            call sync() before starting a run
    -t, --tid <tid>       stat events on existing thread id
```

示例
前面统计程序的示例，下面看一下统计CPU信息的示例：

执行sudo perf stat -C 0，统计CPU 0的信息。想要停止后，按下Ctrl+C终止。可以看到统计项一样，只是统计对象变了。
```
~/perf$ sudo perf stat -C 0
  Performance counter stats for 'CPU(s) 0':

       2517.107315      cpu-clock (msec)          #    1.000 CPUs utilized          
              2,941      context-switches          #    0.001 M/sec                  
                109      cpu-migrations            #    0.043 K/sec                  
                 38      page-faults               #    0.015 K/sec                  
        644,094,340      cycles                    #    0.256 GHz                      (49.94%)
         70,425,076      stalled-cycles-frontend   #   10.93% frontend cycles idle     (49.94%)
        965,270,543      stalled-cycles-backend    #  149.86% backend cycles idle      (49.94%)
        623,284,864      instructions              #    0.97  insn per cycle         
                                                   #    1.55  stalled cycles per insn  (50.06%)
         65,658,190      branches                  #   26.085 M/sec                    (50.06%)
          3,276,104      branch-misses             #    4.99% of all branches          (50.06%)

       2.516996126 seconds time elapsed
```
如果需要统计更多的项，需要使用-e，如：
```
perf stat -e task-clock,context-switches,cpu-migrations,page-faults,cycles,stalled-cycles-frontend,stalled-cycles-backend,instructions,branches,branch-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-loads,dTLB-load-misses ls
结果如下，关注的特殊项也纳入统计。

~/perf$ sudo perf stat -e task-clock,context-switches,cpu-migrations,page-faults,cycles,stalled-cycles-frontend,stalled-cycles-backend,instructions,branches,branch-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-loads,dTLB-load-misses ls
Performance counter stats for 'ls':

          2.319422      task-clock (msec)         #    0.719 CPUs utilized          
                  0      context-switches          #    0.000 K/sec                  
                  0      cpu-migrations            #    0.000 K/sec                  
                 89      page-faults               #    0.038 M/sec                  
          2,142,386      cycles                    #    0.924 GHz                    
            659,800      stalled-cycles-frontend   #   30.80% frontend cycles idle   
            725,343      stalled-cycles-backend    #   33.86% backend cycles idle    
          1,344,518      instructions              #    0.63  insn per cycle         
                                                   #    0.54  stalled cycles per insn
      <not counted>      branches                                                    
      <not counted>      branch-misses                                               
      <not counted>      L1-dcache-loads                                             
      <not counted>      L1-dcache-load-misses                                       
      <not counted>      LLC-loads                                                   
      <not counted>      LLC-load-misses                                             
      <not counted>      dTLB-loads                                                  
      <not counted>      dTLB-load-misses                                           

       0.003227507 seconds time elapsed
```

## 火焰图相关命令

### 生成火焰图

```
perf record -C 0 -F 99 -g -- sleep 30
perf script > out.perf
```

### 折叠调用栈
```
FlameGraph/stackcollapse-perf.pl out.perf > out.folded
```

### 生成火焰图
```
FlameGraph/flamegraph.pl out.folded > out.svg
```



