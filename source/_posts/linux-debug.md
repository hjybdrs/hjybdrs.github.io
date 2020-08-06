---
title: linux-debug
date: 2020-07-29 14:49:57
tags:
---

# Linux-Debug

## coredump

系统默认不会生成core 文件，依赖shell 中的设置。
```shell
$ ulimit -a
# -c: core file size (blocks)         0 这一行表示不会有core 文件生成
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-v: address space (kbytes)          unlimited
-l: locked-in-memory size (kbytes)  unlimited
-u: processes                       1392
-n: file descriptors                10240

# 允许生成core 文件，文件大小不受限
$ ulimit -c unlimited

# 默认core 和可执行文件在同一目录，但可以设置
# /proc/sys/kernal/core_pattern 可以格式化core 文件路径和文件名

```

## performance

### perf

```shell
# 可以使用/ 进行某些关键字查询
$ perf top
# 单独的监控某一个应用程序
$ perf stats xxx.exe
```
https://juejin.im/post/6844903950315945992
http://linux.51yip.com/search/perf