# linux内核分析及应用 读书笔记
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


