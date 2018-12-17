---
layout:     post
title:      Git 回退提交方法
subtitle:   Git 回退提交方法
date:       2018-12-17
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Git
---

# 1. 场景重现
## 1.1. 本地新建feature01完成功能开发
```
$ git checkout master   #从master分支创建新分支，如果当前在其他分支，先切换到master
$ git pull              #拉取最新版本
$ git checkout -b feature01    #从master创建分支，并切换到新分支
Switched to a new branch 'feature01'
```

新建2个提交完成特性开发：
commit1:
```
$ mkdir commit1
$ touch commit1/commit1_change.txt
$ vi commit1/commit1_change.txt
This is commit1/commit1_change.txt changes.

$ git status
On branch feature01
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	commit1/

nothing added to commit but untracked files present (use "git add" to track)

$ git add commit1/

$ git commit -m "commit1 comment"
[feature01 3c9bda6] commit1 comment
 1 file changed, 1 insertion(+)
 create mode 100644 commit1/commit1_change.txt
```
同理，完成commit2.
提交合并回master分支，并推送到远程仓库：
```
$ git branch 
* feature01
  master

$ git checkout master

$ git merge feature01 
Updating 959af4f..84950bf
Fast-forward
 commit1/commit1_change.txt | 1 +
 commit2/commit2_change.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 commit1/commit1_change.txt
 create mode 100644 commit2/commit2_change.txt

$ git push origin master
```
上面的步骤完成了一个特性的开发，此时可以删除 本地feature01 分支 （本地分支可以任意多个，只要不推送到远程都没有关系）。

## 1.2. 本地新建feature02完成功能开发
新建2个提交完成特性开发：commit3, commit4.
参见1.1节。

## 1.3. 紧急修复分支hotfix
此时发现feature01提交的第二次commit ID  84950bfc2976fd7595df8a81cc6453d8d69354c6  产生了一个bug需要修复。如果是在该提交上修复，则直接从 commit ID 84950bfc2976fd7595df8a81cc6453d8d69354c6 新建一个分支 hotfix 来解决这个bug；如果是撤销本次改动，则从该commit ID的前一个ID新建一个分支hotfix。
```
$ git checkout -b hotfix 84950bfc2976fd7595df8a81cc6453d8d69354c6
Switched to a new branch 'hotfix'

```
在hotfix上fix bug，并提交，commit ID为：22dbafa5dfeb3723ac49319d974d6573c83434d0
切换到master，并merge hotfix上的新提交：
```
$ git checkout master 
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.

$ git merge hotfix 
Merge made by the 'recursive' strategy.
 commit2/commit2_change.txt | 1 +
 1 file changed, 1 insertion(+)

```
merge时，会自动将master上的commit3和commit4合并过来：
```
git log
commit 520786d6ba6ca27ce5725922f56c608187152bd9
Merge: 1f8cdf1 22dbafa
Author: naiquan.hu 
Date:   Mon Dec 17 12:26:20 2018 +0800

    Merge branch 'hotfix'

commit 22dbafa5dfeb3723ac49319d974d6573c83434d0
Author: naiquan.hu 
Date:   Mon Dec 17 12:23:43 2018 +0800

    patch for commit2

commit ba286e16e465c9a3134ab1ce8fd974029163feb1
Author: naiquan.hu
Date:   Mon Dec 17 12:03:05 2018 +0800

    commit4 comment

commit aa26c9efda7ded98d640e4705a4a587da1c5935d
Author: naiquan.hu
Date:   Mon Dec 17 12:01:50 2018 +0800

    commit3 comment
```
推送到远程仓库：
```
$ git push origin master
```

# 2. 文档版本打标签
给当前的master分支打标签v0.1，然后推送到远程:
```
$ git tag -m "create tag v0.1 for master" v0.1
$ git push origin --tags
```

# 2. 回滚远程厂库
本地master回退到指定版本：aa26c9efda7ded98d640e4705a4a587da1c5935d
```
$ git reset --hard aa26c9efda7ded98d640e4705a4a587da1c5935d
HEAD is now at aa26c9e commit3 comment
```
把本地master 强制推送到远程分支:
```
$ git push origin master -f
```
注意： 这样回滚后，远程厂库里的commit3之后的所有commit都会没有了。