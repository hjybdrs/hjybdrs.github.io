---
title: PerfCommand
date: 2020-08-16 22:50:53
tags:
hide: true
---


perf 基本原理是在cpu PMU(performance Monitor Unit) 寄存器和软件特性中获取相关信息来进行系统性能输出。
可以监控的事件分为三类:
* Hardware Event 这块主要是由PMU 寄存机硬件产生的，如如cache miss 等，如果需要了解程序对硬件特性使用情况，需要使用这类事件
* Software Event 内核软件产生的时间，比如进程切换  tick 数
* Tracepoint Event 内核中的静态tracepoint 触发的时间，这些tracepoint 用来判断程序运行期间内核的行为细节，比如slab 分配器的分配次数等
```shell
# 可以使用/ 进行某些关键字查询
$ perf top
# 单独的监控某一个应用程序
# 事件这么多，不知道监控哪一些，无从下手，最好先从整体到细节。先大致了解有哪一些统计事件，在到具体细节中去
# 使用perf stat 可以通过精简的方式提供程序运行的整体情况和数据汇总
$ perf stats xxx.exe
# task-clock 指的是占用了多少任务时钟周期，越高说明多数时间话费在CPU 计算上

$ perf record -g -e kmem:* -F 99 -p 12413 -- sleep 20
$ perf record -g -e kmem:* -F 99 command
$ perf kmem -i perf.data --caller stat
# perf 生成火焰图
FlameGraph/stackcollapse-perf.pl result.perf > result.folded
FlameGraph/flamegraph.pl result.folded > cpu.svg

diff 火焰图

除了通常的几种火焰图，我们其实还可以将两个火焰图进行 diff，生成一个 diff 火焰图，

./FlameGraph/difffolded.pl perf.folded perf.folded1 | ./flamegraph.pl > ../diff.svg
```
https://juejin.im/post/6844903950315945992
http://linux.51yip.com/search/perf
https://blog.csdn.net/qq_15437667/article/details/50724330
https://www.cnblogs.com/sunsky303/p/8328836.html
http://www.brendangregg.com/perf.html