---
title: Linux-Mem
date: 2020-08-31 14:42:11
tags:
---


# Linux-Mem

## 什么是buffer/cache

buffer 指的是buffer cache，中文语境下为：缓冲区缓存，历史上主要是对io 设备写的缓存
cache  指的是page cache，中文语境下为：页面缓存，历史上主要是对io 设备读的缓存

但是目前，他们两个的意义已经不一样了，如果内存是以page 进行分配管理的，都可以使用page cache 作为其缓存来管理使用。当然并不是所有的内存都是以page 来管理的，还有按照block 来进行管理的，一般叫做buffer。