---
title: git 常用指令
date: 2020-03-28 10:02:29
tags: 
    - git
---

# 指令

## 仓库操作

### 克隆远端仓库

```shell
# 在本地生成一个目录
git clone gitxxxxURL.git

# 在本地生成一个指定目录
git clone gitxxxxURL.git myGitDir

# clone 远端仓库的指定分支
git clone -b branchName gitxxxxURL.git

```

### 分支推送

```shell
# 将本地当前分支推送到远端指定分支，远端分支不需要加origin 因为前面的origin 已经代表了绑定的远端仓库
git push origin HEAD:specificBranch
# 本地的分支强制更新远端的指定分支，远端分支的内容有可能被覆盖
git push origin HEAD:specificBranch --force
# 将远端指定分支删除
git push origin :specificBranch
```



## 分支操作

### 跟踪远程指定分支

跟踪后可以直接pull 和push，不需要指定对应的分支名

```shell
# 生成一个和远端分支名一样的本地分支
git checkout --track origin/branchName
# 生成一个本地指定分支与远端分支关联
git checkout -b localBranchName origin/branchName
# 从指定的提交号中生成一个本地分支
git checkout -b localBranchName commit_id
```

## 提交处理

### cherry-pick

```shell
# 这样会生成一个新的commit-id，如果有冲突的话，还需要解决冲突 
git cherry-pick commit-id 
# 中间是两个点.. 
# 提交一个范围内的commit-id 左开右闭 
git cherry_pick <start-commit-id>..<end-commit-id> 
# 提交一个范围内的commit-id 左闭右闭 
git cherry_pick <start-commit-id>^..<end-commit-id> 
```



## 密钥生成

```shell
ssh-keygen -t rsa -b 4096 -C "handsom@guy.com" 
```

