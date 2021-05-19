# 软件环境
CentOS Linux release 8.3.2011 

# 按照libasan包
yum install libasan

# 测试例子
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
 
int my_print()
{
    int *a = malloc(10);
    printf("this is my_print, a:%p\n", a);
    return 0;
}
 
int main()
{
    my_print();
    return 0;
}
```

# 编译连接libasan
gcc -g tmp.c -lasan -o hello -fsanitize=address -fno-omit-frame-pointer -fsanitize-recover=address

# 运行例子
```
[sangfor@sfos-x86_64 ych]# ./hello 
this is my_print, a:0x602000000010

=================================================================
==3487654==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 10 byte(s) in 1 object(s) allocated from:
    #0 0x7f209308fba8 in __interceptor_malloc (/lib64/libasan.so.5+0xefba8)
    #1 0x4007d7 in my_print /var/ych/tmp.c:7
    #2 0x400806 in main /var/ych/tmp.c:14
    #3 0x7f2092c007b2 in __libc_start_main (/lib64/libc.so.6+0x237b2)

SUMMARY: AddressSanitizer: 10 byte(s) leaked in 1 allocation(s).
```

# 指定asan选项启动进程
ASAN_OPTIONS=halt_on_error=0:user_sigaltstack=0:detect_leaks=1:malloc_context_size=15:log_path=/var/ych/asan.log ./hello
```
[sangfor@sfos-x86_64 ych]# ASAN_OPTIONS=halt_on_error=0:user_sigaltstack=0:detect_leaks=1:malloc_context_size=15:log_path=/var/ych/asan.log ./hello
this is my_print, a:0x602000000010

[sangfor@sfos-x86_64 ych]# cat asan.log.3488778 

=================================================================
==3488778==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 10 byte(s) in 1 object(s) allocated from:
    #0 0x7fd46ca72ba8 in __interceptor_malloc (/lib64/libasan.so.5+0xefba8)
    #1 0x4007d7 in my_print /var/ych/tmp.c:7
    #2 0x400806 in main /var/ych/tmp.c:14
    #3 0x7fd46c5e37b2 in __libc_start_main (/lib64/libc.so.6+0x237b2)

SUMMARY: AddressSanitizer: 10 byte(s) leaked in 1 allocation(s).
```

# 相关链接

[asan官方资料](https://github.com/google/sanitizers/wiki/AddressSanitizer)

[AddressSanitizerFlags](https://github.com/google/sanitizers/wiki/AddressSanitizerFlags)