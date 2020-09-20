---
title: LinuxCpu
date: 2020-08-13 09:37:47
tags:
hide: true
---



# CPU

## 如何查看CPU 数量

一般来说，一颗物理CPU 上可以分出多个逻辑核心，加上intel 的超线程技术，可以在逻辑核心上再分出一倍数量的核心出来。

```
# 如果是相同physical id 的cpu，那么是同一个物理核心封装的物理核或者超线程
# 如果是相同core id 的cpu，那么是同一个逻辑核的超线程
$ cat /proc/cpuinfo

# 查看有多少个物理核心
$ cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l

#查看有多少个逻辑核心
$ cat /proc/cpuinfo | grep "processor id" | wc -l
```

## CPU 平均负载

top 和 uptime 命令都可以查看

系统负载升高的原因:
一般来说，系统平均负载升高意味着CPU 利用率上升，但是没有必然联系。
对于CPU 密集型的操作来说平均负载上升，一般意味着CPU 利用率上升。
如果是IO 密集型任务较多的话，此事的CPU 利用率不一定高，可能很多任务处于不可中断状态，等待CPU的调度也会升高平均负载。

所以在遇到系统平均负载很高，但是CPU利用率上不去的时候，就需要考虑系统是否遇到了IO 瓶颈。
因此系统是否遇到CPU瓶颈需要结合CPU 利用率，系统平均负载来查看。

https://community.tenable.com/s/article/What-is-CPU-Load-Average
https://scoutapm.com/blog/understanding-load-averages
https://www.ruanyifeng.com/blog/2011/07/linux_load_average_explained.html