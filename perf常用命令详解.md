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



