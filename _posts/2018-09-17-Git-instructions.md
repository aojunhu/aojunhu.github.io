---
layout:     post
title:      Git使用说明
subtitle:   Git使用说明
date:       2018-09-17
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
    - Git
---

说明：使用人员请直接阅读第三节。
# 1. Git服务器端配置
说明：服务器端已配置好，使用人员无需关注本节。
## 1.1. 安装 openssh服务器

```
sudo apt-get install openssh-server openssh-client
```

## 1.2. 安装 git server

```
sudo apt-get install git-core
```

## 1.3. 配置 git server
创建服务器管理用户git:

```
sudo useradd -m git
sudo passwd git
```
更改git用户默认目录，设置权限：

```
sudo mkdir /projects/art2000/git/git_projects
sudo chown git:git /projects/art2000/git/git_projects
sudo chmod 755 /projects/art2000/git/git_projects
sudo vi /etc/passwd
git:x:1001:1001::/projects/art2000/git/git_projects:
```
## 1.4. Git账户生成管理密钥

```
su git
ssh-keygen -t rsa
```
# 1.5. 安装python的setup tool

```
sudo apt-get install python-setuptools
```

## 1.6. 获取并安装gitosis

```
cd /tmp
git clone https://github.com/res0nat0r/gitosis.git
cd gitosis
sudo python setup.py install
```

## 1.7. 配置gitosis

```
su git
gitosis-init < ~/.ssh/id_rsa.pub
sudo chmod 755 /projects/art2000/git/git_projects /repositories/gitosis-admin.git/hooks/post-update
```
gitosis-init命令会在/projects/art2000/git/git_projects下创建gitosis和repositories, repositories是所有仓库的根目录。Repositories下会生成一个用于权限管理的仓库gitosis-admin.git.

## 1.8. 禁用git用户的shell登陆
出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：

```
git:x:5005:5005:git:/home/git:/bin/bash
```
最后一个冒号后改为：

```
git:x:5005:5005:git:/home/git:/usr/bin/git-shell
```
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

# 2. 建立服务器端仓库
说明：使用人员无需关注本节。
## 2.1. 修改gitosis-admin
使用git账户登录，修改gitosis-admin配置文件，增加某用户对新建远程仓库的操作权限。

```
[group developers]
members = userA@server
writable = new_repository
```

## 2.2. 建立远程仓库
以userA登录：
（server_ip=192.168.68.8）

```
mkdir new_repository
git init
git remote add origin git@server_ip:new_repository.git
touch README
git add .
git commit -a -m "initial new_repository"
git push origin master
```

## 2.3. 管理gitosis配置
说明：服务器IP地址为192.168.68.8，故下文所有server_ip都为192.168.68.8.
目前具有gitosis管理权限有三个账号（git@art002, naiquan.hu, xinwei.wu），如有新人需加权限，请联系xinwei.wu。
```
su git
git clone git@192.168.68.8:gitosis-admin.git
cd gitosis-admin/
```
里面有gitosis.conf，keydir。修改gitosis.conf文件，如下所示：

```
[gitosis]

[group gitosis-admin]
writable = gitosis-admin
members = a@server1

[group developers]
writable = helloworld
members = a@server1 b@server2

[group test]
readonly = helloworld
members = c@server3
```
这个配置文件表达了如下含义：gitosis-admin组成员有a，该组对gitosis-admin仓库有读写权限； developers组有a，b两个成员，该组对helloworld仓库有读写权限； test组有c一个成员，对helloworld仓库有只读权限。 当然目前这些配置文件的修改只是在你的本地，你必须推送到gitserver上才能真正生效。 加入新文件、提交并push到git服务器.
例如，给jianwei.yang@lightningsemi.com添加soc.git访问权限：
1.修改gitosis.conf， 把jianwei.yang@lightningsemi.com加入到[group digital]
2. 把jianwei.yang的公钥~/.ssh/id_rsa.puh拷贝到gitosis-admin/keydir，并重命名为jianwei.yang@lightningsemi.com.pub。提交到服务器生效。

# 3. 客户端使用
说明：使用人员只需要关注本节。（以用户nqhu为例）
## 3.1. 创建SSH Key

```
[nqhu@art002 ~]$ ssh-keygen -t rsa -C "user_name@email.com"
[nqhu@art002 ~]$ cp ~/.ssh/id_rsa.pub /tmp/user_name@email.com.pub
```
联系git管理员加入到gitosis-admin，参见2.3小结。

## 3.2. clone仓库test
```
[nqhu@art002 ~]$ git clone git@192.168.68.8:test.git test
```

## 3.3. 增加文件和目录

```
[nqhu@art002 ~]$ cd test
[nqhu@art002 test]$ mkdir newfolder
[nqhu@art002 test]$ cd newfolder
[nqhu@art002 newfolder]$ vi test.txt
Input content and save:
Add this for test.
```

## 3.4. 查看修改情况
```
[nqhu@art002 ~]$ git status
```

## 3.5. 添加目录newfolder及test.txt
```
[nqhu@art002 test]$ git add newfolder/test.txt
```

## 3.6. 配置git commit参数
```
[nqhu@art002 test]$ git config --global core.editor vim
[nqhu@art002 test]$ git config --global user.name "naiquan.hu"
[nqhu@art002 test]$ git config --global user.email  "naiquan.hu@email.com"
```
## 3.7. 提交修改

```
[nqhu@art002 test]$ git commit -m "commit message"
```
## 3.8. 推送至远程仓库test.git
```
[nqhu@art002 test]$ git push origin master
```

## 3.9. 查看历史记录
```
[nqhu@art002 test]$ git log
```

## 3.10. 更新代码
```
[nqhu@art002 test]$ git pull
```

## 3.11. 再次修改newfolder/test.txt并查看改动内容
```
[nqhu@art002 test]$ vi newfolder/test.txt
add:
Change this second time.
[nqhu@art002 test]$ git diff newfolder/test.txt
```

## 3.12. 再次提交并推送至远程仓库
```
[nqhu@art002 test]$ git add newfolder/test.txt
[nqhu@art002 test]$ git commit
[nqhu@art002 test]$ git push origin master
```

## 3.13. 查看记录
```
[nqhu@art002 test]$ git log
```

## 3.14. 查看前后两个版本的差异
```
[nqhu@art002 test]$ git diff commit-id-x commit-id-y
```

## 3.15. 回退到某一个commit
```
[nqhu@art002 test]$ git reset --hard commit-id-x
```

## 3.16. 回退掉某一个commit
比如我的提交历史如下，我现在想删除commit_B，但是不影响commit_B之后的提交历史。
```
commit_C 

commit_B

commit_A
```
执行如下命令：
```
git rebase -i  commit_A
```
将commit_B这一行前面的pick改为drop，然后按照提示保存退出。至此已经删除了指定的commit，可以使用git log查看下。
推送至远程仓库：
```
git push origin HEAD –force
```
## 3.17. 查看commit提交记录详情
1. 查看最新的commit
```
git show
```
2. 查看指定commit hashID的所有修改：
```
git show commitId
```
3. 查看某次commit中具体某个文件的修改：
```
git show commitId fileName
```

## 3.18. git stash 用法

常用git stash命令：

1. git stash save "save message" 
执行存储时，添加备注，方便查找，只有git stash 也要可以的，但查找时不方便识别。

2. git stash list  
查看stash了哪些存储

3. git stash show
显示做了哪些改动，默认show第一个存储,如果要显示其他存贮，后面加stash@{$num}，比如第二个 git stash show stash@{1}

4. git stash show -p 
显示第一个存储的改动，如果想显示其他存存储，命令：git stash show  stash@{$num}  -p ，比如第二个：git stash show  stash@{1}  -p

5. git stash apply 
应用某个存储,但不会把存储从存储列表中删除，默认使用第一个存储,即stash@{0}，如果要使用其他个，git stash apply stash@{$num} ， 比如第二个：git stash apply stash@{1} 

6. git stash pop
命令恢复之前缓存的工作目录，将缓存堆栈中的对应stash删除，并将对应修改应用到当前的工作目录下,默认为第一个stash,即stash@{0}，如果要应用并删除其他stash，命令：git stash pop stash@{$num} ，比如应用并删除第二个：git stash pop stash@{1}

7. git stash drop stash@{$num} 
丢弃stash@{$num}存储，从列表中删除这个存储

8. git stash clear
删除所有缓存的stash

实例：
查看改动：
```
$ git status
On branch develop
Your branch is up to date with 'origin/develop'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx
        modified:   project/firmware_XIP/usr_app/usr_app.c
        modified:   src/wifi/wifi_mac/Controller/AP-STA/iconfig.c

no changes added to commit (use "git add" and/or "git commit -a")
```
这些改动暂时不需要提交到服务器，但更新代码时可能会有冲突，所以需要暂存起来，命令如下：
```
$ git stash save "save this files for next using"
warning: LF will be replaced by CRLF in project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx.
The file will have its original line endings in your working directory
Saved working directory and index state On develop: save this files for next using
```
再次查看，已经看不到改动，更新代码就不会有冲突了：
```
$ git status
On branch develop
Your branch is up to date with 'origin/develop'.

nothing to commit, working tree clean
```

查看存储：
```
$ git stash list
stash@{0}: On develop: save this files for next using
```

显示做了哪些改动:
```
$ git stash show
 project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx | 2 +-
 project/firmware_XIP/usr_app/usr_app.c                | 5 +++++
 src/wifi/wifi_mac/Controller/AP-STA/iconfig.c         | 4 +---
 3 files changed, 7 insertions(+), 4 deletions(-)

```

显示第一个存储的改动:
```
$ git stash show -p
diff --git a/project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx b/project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx
index e4b33e3..ea9705c 100644
--- a/project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx
+++ b/project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx
@@ -82,7 +82,7 @@
             <RunUserProg1>1</RunUserProg1>
             <RunUserProg2>1</RunUserProg2>
             <UserProg1Name>python ..\..\..\tools\user_cmd\after_build_soc.py firmware_XIP</UserProg1Name>
-            <UserProg2Name>..\..\..\tools\bin\mkimage.exe ln881x flashimage ..\..\..\lib\boot_ram_ln881x.bin firmware_XIP.bin flashimage.bin release=1 crp_enable=0 app_version=10 hw_version=0</UserProg2Name>
+            <UserProg2Name>..\..\..\tools\bin\mkimage.exe ln881x flashimage ..\..\bootload\bootram\keil_ln881x\boot_ram.bin firmware_XIP.bin flashimage.bin release=0 crp_enable=0 app_version=10 hw_version=0</UserProg2Name>
             <UserProg1Dos16Mode>0</UserProg1Dos16Mode>
             <UserProg2Dos16Mode>0</UserProg2Dos16Mode>
             <nStopA1X>0</nStopA1X>
diff --git a/project/firmware_XIP/usr_app/usr_app.c b/project/firmware_XIP/usr_app/usr_app.c
index 7cb2715..61b58c6 100644
--- a/project/firmware_XIP/usr_app/usr_app.c
+++ b/project/firmware_XIP/usr_app/usr_app.c
@@ -20,11 +20,15 @@ static OS_Thread_t g_usr_app2_thread;
 void wifi_init_sta(void)
 {
     wifi_config_t wifi_config = {
+#if 0
         .sta = {
             .ssid     = "Tenda_444",
             .password = "12345678",
             0,
         },
+#else
+    0
+#endif
     };

```
再次使用这些stash，直接使用git stash apply或者git stash pop就可以再次导出来了。git stash apply不会把存储从存储列表中删除，git stash pop将缓存堆栈中的对应stash删除。
```
$ git stash pop
On branch develop
Your branch is up to date with 'origin/develop'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   project/firmware_XIP/keil_ln881x/firmware_XIP.uvprojx
        modified:   project/firmware_XIP/usr_app/usr_app.c
        modified:   src/wifi/wifi_mac/Controller/AP-STA/iconfig.c

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (c54638d9a8ca0a6a8ebb9ccc2d1025fef1ceba89)

```