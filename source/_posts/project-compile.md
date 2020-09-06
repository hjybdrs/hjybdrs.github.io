---
title: project-compile
date: 2020-09-04 14:41:47
tags:
    - webrtc
    - chromuim
---

# 项目编译

## Requirements

```shell
# depot_tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

# export tools
export PATH="$PATH:/path/to/depot_tools"

# fetch
# 查看哪些项目代码可以获取
fetch --help
# 获取对应代码
fetch webrtc
```

## Webrtc
```shell
# 创建文件夹 拉取代码  同步代码
mkdir webrtc-checkout && cd webrtc-checkout && fetch webrtc && gclient sync
# 生成ninja project
gn gen out/Default
# 编译
ninja -C out/Default

# https://webrtc.github.io/webrtc-org/native-code/development 
```

## Chromuim
```shell
# 创建文件夹 拉取代码
mkdir chrome && cd chrome && fetch chromium
# 生成ninja project
gn gen out/Default
# 编译
ninja -C out/Default

# 生成的可执行文件在 Chromium.app/Contents/MacOS
# https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md
```