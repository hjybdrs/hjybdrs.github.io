---
title: DevelopeKit
date: 2020-08-04 14:31:47
tags:
hide: true
---

# DevelopeKits


## Command

### z.sh

[zsh](https://github.com/rupa/z.git)

### tldr

[tldr](https://github.com/tldr-pages/tldr)

### zip

```shell
# 压缩打包一个文件夹
zip -r dst.zip folder
# 压缩打包一个文件
zip dst.zip file
```

### 把数据同时输出到文件和屏幕

```shell
./happy_day | tee log.txt | cat
```

### 重定向

```shell
#表明把标准输出重定向到文件 
./happy_day > file

#把标准错误输出重定向到标准输出
./happy_day 2>&1 file
./happy_day &> file
```

## grep 和find 配合

```shell
find ./dir -name "*cpp" | xargs grep "xxxx"
# 查找文件 -type f
find ./dir -type f | xargs grep "xxxx"
```