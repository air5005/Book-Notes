# gdb常用命令
1. list(简写 l)： 查看源程序代码，默认显示10行，按回车键继续看余下的
2. run(简写 r) ：运行程序直到遇到 结束或者遇到断点等待下一个命令
3. break(简写 b) ：格式 b 行号，在某行设置断点
4. info breakpoints ：显示断点信息
5. 单步调试
	使用 continue、step、next命令
6. 查看变量
	使用print、whatis命令
7. info args：查看当前函数的参数
8. 多线程调试
	info threads : 查看所有线程
	thread n : 跳到另外一个线程
9. info register：查看寄存器信息

# gdb之x命令
	可以使用examine命令(简写是x)来查看内存地址中的值。x命令的语法如下所示：
Examine memory: x/FMT ADDRESS.
x/<n/f/u> <addr>
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
t(binary), f(float), a(address), i(instruction), c(char) and s(string).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).

x /10xg addr

# 使用命令查看程序内部参数变量
echo 'p malloc_stats_print(0,0,0)' | gdb --quiet -nx -p 24555

# 设置gdb的coredump_filter
https://man7.org/linux/man-pages/man5/core.5.html

/proc/[pid]/coredump_filter file can be used to control which
       memory segments are written to the core dump file in the event
       that a core dump is performed for the process with the
       corresponding process ID.

       The value in the file is a bit mask of memory mapping types (see
       mmap(2)).  If a bit is set in the mask, then memory mappings of
       the corresponding type are dumped; otherwise they are not dumped.
       The bits in this file have the following meanings:

           bit 0  Dump anonymous private mappings.
           bit 1  Dump anonymous shared mappings.
           bit 2  Dump file-backed private mappings.
           bit 3  Dump file-backed shared mappings.
           bit 4 (since Linux 2.6.24)
                  Dump ELF headers.
           bit 5 (since Linux 2.6.28)
                  Dump private huge pages.
           bit 6 (since Linux 2.6.28)
                  Dump shared huge pages.
           bit 7 (since Linux 4.4)
                  Dump private DAX pages.
           bit 8 (since Linux 4.4)
                  Dump shared DAX pages.

echo 0xff > /proc/$(pidof worker)/coredump_filter
