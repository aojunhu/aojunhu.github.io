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
