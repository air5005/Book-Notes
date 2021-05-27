[TOC]



# perf常用定位性能瓶颈命令

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

用perf sched 是一个针对cpu调度的专用命令，可以用于分析cpu调度器的行为，查看调度延迟和唤醒延迟

```
perf sched record -C 1 sleep 10       # 记录指定cpu指定时间段内的调度事件
perf sched latency                    # 查看调度时延
perf sched script                     # 查看调度详情
perf sched timehist                   # 查看任务调度详细时间， 包括休眠时间、调度器延时时间、cpu运行时长等
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



## 火焰图相关命令

### 生成火焰图

```
git clone https://github.com/brendangregg/FlameGraph.git
cd FlameGraph
perf record -F 99 -ag -- sleep 10
perf script --header | ./stackcollapse-perf.pl | ./flamegraph.pl > flame.svg
```

stackcollapse-perf.pl 将perf script的输出转化为 flamegraph.pl 可以接受的标准格式。 FlameGraph 代码仓库中有很多其他性能分析器的输出转化脚本。flamegraph.pl程序生成了一个SVG文件格式的火焰图，自带javascript以便在浏览器中交互，该脚本有很多定制选项， -help可以查看。

## 定位系统是否存在大量短进程的方法

perf stat -e sched:sched_process_exec -I 1000

```
[sangfor@sfos-x86_64 FlameGraph]# perf stat -e sched:sched_process_exec -I 1000
#           time             counts unit events
     1.000142123                 10      sched:sched_process_exec
     2.000588175                  1      sched:sched_process_exec
     3.000748017                 15      sched:sched_process_exec
     4.000922860                  4      sched:sched_process_exec
     5.001087477                 15      sched:sched_process_exec
     6.001253585                  1      sched:sched_process_exec
     7.001428253                 10      sched:sched_process_exec
     8.001597555                  3      sched:sched_process_exec
     9.001759195                 10      sched:sched_process_exec
    10.004985643                 92      sched:sched_process_exec
```

通过上面可以看到 exec 每秒在15次左右，最后一次由 92 个进程创建，可以增加 -C 来缩小进程在哪些cpu运行的来缩小范围。

怎么进一步查看exec命令的详细信息：

perf record -e sched:sched_process_exec -a

perf script

```
[sangfor@sfos-x86_64 ych]# perf script
           sleep 1295854 [000] 523461.266349: sched:sched_process_exec: filename=/usr/bin/sleep pid=1295854 old_pid=1295854
              sh 1295855 [000] 523462.110869: sched:sched_process_exec: filename=/bin/sh pid=1295855 old_pid=1295855
              wc 1295856 [000] 523462.113759: sched:sched_process_exec: filename=/usr/bin/wc pid=1295856 old_pid=1295856
             awk 1295857 [000] 523462.115227: sched:sched_process_exec: filename=/usr/bin/awk pid=1295857 old_pid=1295857
              sh 1295858 [000] 523462.129435: sched:sched_process_exec: filename=/bin/sh pid=1295858 old_pid=1295858
              wc 1295859 [000] 523462.132767: sched:sched_process_exec: filename=/usr/bin/wc pid=1295859 old_pid=1295859
             awk 1295860 [000] 523462.133977: sched:sched_process_exec: filename=/usr/bin/awk pid=1295860 old_pid=1295860
              sh 1295861 [000] 523462.141481: sched:sched_process_exec: filename=/bin/sh pid=1295861 old_pid=1295861
              wc 1295862 [000] 523462.145605: sched:sched_process_exec: filename=/usr/bin/wc pid=1295862 old_pid=1295862
             awk 1295863 [000] 523462.147583: sched:sched_process_exec: filename=/usr/bin/awk pid=1295863 old_pid=1295863
              sh 1295864 [000] 523462.163731: sched:sched_process_exec: filename=/bin/sh pid=1295864 old_pid=1295864
              wc 1295865 [000] 523462.166772: sched:sched_process_exec: filename=/usr/bin/wc pid=1295865 old_pid=1295865
             awk 1295866 [000] 523462.167886: sched:sched_process_exec: filename=/usr/bin/awk pid=1295866 old_pid=1295866
           sleep 1295867 [000] 523462.269520: sched:sched_process_exec: filename=/usr/bin/sleep pid=1295867 old_pid=1295867
 area_ctl_sip_re 1295868 [000] 523462.445619: sched:sched_process_exec: filename=/sfos/system/app/af/bin/area_ctl_sip_read pid=1295868 old_pid=1295868

```

使用以上命令可以看出来，每个短进程的详细信息了。


