[TOC]

------

# nginx完全开发指南读书笔记

## nginx的特点

三高一低是nginx的主要特点

- 高性能

  C语言编写，采用事件驱动模型，可以无阻塞的处理海量并发连接

- 高稳定性

  内存池、模块化架构、主从进程管理

- 低资源消耗

  没有进程或线程的切换、使用accept4来减少内核调用的次数、使用writev集中发送数据、使用字符串引用而不是拷贝等等

- 高扩展性

  模块化定制开发，高可扩展性

## nginx的配置

1. nginx通过编译增加 --with-stream 参数，能启动stream模块，让nginx能够工作在四层网络上，直接处理TCP/UDP协议

2. 在conf配置文件里面增加 accept_mutex on|off，是否启动进程间的负载均衡机制，但是也引入了锁机制，影响效率。

   现在最新的nginx还可以使用内核级别的负载均衡技术，如 EPOLLEXCLUSIVE 和 REUSEPORT

# 相关框架图

## 进程uml

![nginx_process](D:\work\code\annotated_nginx\diagrams\nginx_process.jpg)

## stream机制uml

![ngx_stream](D:\work\code\annotated_nginx\diagrams\ngx_stream.jpg)

