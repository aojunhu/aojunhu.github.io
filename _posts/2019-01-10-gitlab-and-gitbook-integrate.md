---
layout:     post
title:      GitLab安装以及与GitBook集成
subtitle:   GitLab安装以及与GitBook集成
date:       2019-01-10
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
- Git
- GitLab
- GitBook
---

# 1 GitLab安装与配置

官网：https://about.gitlab.com/installation/
官网有针对各系统的安装教程。
## 1.1 安装依赖包
```
sudo apt-get install curl openssh-server ca-certificates postfix
```

执行完成后，出现邮件配置，选择Internet那一项（不带Smarthost的）

## 1.2 下载GitLab安装包
```
https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/trusty/gitlab-ce_11.6.3-ce.0_amd64.deb/download.deb
```

## 1.3 安装
```
sudo -i gitlab-ce_11.6.3-ce.0_amd64.deb
```
执行完成后，在执行下面的命令：
```
sudo gitlab-ctl  reconfigure
```

## 1.4 验证是否安装和启动成功
```
sudo gitlab-ctl  status
```

还可以直接在浏览器里来访问GitLab web系统:
```
http://server_ip:80
```

## 1.5 管理员登录
```
user_name: root
password: xxx
```
登录后可以创建组、用户、项目等内容。比如创建组software，用户gitlab_test

## 1.6 GitLab本地配置
GitLab包含了众多插件，要配置端口等信息，打开/etc/gitlab/gitlab.rb：
```
sudo vi /etc/gitlab/gitlab.rb
```
**配置外部URL，Nginx端口，Unicorn端口**
默认情况下，外部URL为 'http://gitlab.example.com' ，这个域名外部是无法访问的，除非你做了映射，如果不想做什么映射，直接将该域名改成本机IP:Nginx_port
```
#change port
unicorn['listen'] = '127.0.0.1'
unicorn['port'] = 8090
nginx['listen_addresses'] = ['*', '[::]']
nginx['listen_port'] = 85

external_url 'http://192.168.122.95:85'
```
修改unicorn.rb文件
```
sudo vi /var/opt/gitlab/gitlab-rails/etc/unicorn.rb

# What ports/sockets to listen on, and what options for them.
listen "127.0.0.1:8090", :tcp_nopush => true                                    
listen "/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket", :backlog => 1024
```

**修改邮件配置**
```
sudo vi /etc/gitlab/gitlab.rb

# gitlab_rails['smtp_enable'] = true
# gitlab_rails['smtp_address'] = "smtp.server"
# gitlab_rails['smtp_port'] = 465
# gitlab_rails['smtp_user_name'] = "smtp user"
# gitlab_rails['smtp_password'] = "smtp password"
# gitlab_rails['smtp_domain'] = "example.com"
# gitlab_rails['smtp_authentication'] = "login"
# gitlab_rails['smtp_enable_starttls_auto'] = true
# gitlab_rails['smtp_tls'] = false

### Email Settings
# gitlab_rails['gitlab_email_enabled'] = true
# gitlab_rails['gitlab_email_from'] = 'example@example.com'                     
# gitlab_rails['gitlab_email_display_name'] = 'Example'
# gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
# gitlab_rails['gitlab_email_subject_suffix'] = ''
```

**保存配置**
配置文件修改后，保存不是立刻生效的，还需要命令去修改：
```
sudo gitlab-ctl  reconfigure 
```

**关闭防火墙**
如果系统还开了防火墙，可能会导致后台访问不到，要关闭防火墙，或者设置过滤端口，以下选择关闭防火墙比较方便，如果选择端口设置，则需要看哪个端口要使用，再去设置.
```
service iptables stop
```

## 1.7 GitLab后台配置
本地配置完成后，还需要登录后台配置。打开gitlab 后台，后台地址为：
```
http://gitlab_url:nginx_port
(http://192.168.122.95:85)
```
配置本地网络可访问，配置地址为:
```
http://192.168.122.95:85/admin/application_settings
```
一般无需更改设置。
创建的仓库默认存放在GitLab服务器上的位置：
```
/var/opt/gitlab/git-data/repositories/
```
# 2 安装GitLab Runner
## 2.1 安装
Debian/Ubuntu/CentOS 的安装方法，其他系统去参考官方文档：
```
# For Debian/Ubuntu
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash
$ sudo apt-get install gitlab-ci-multi-runner

# For CentOS
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
$ sudo yum install gitlab-ci-multi-runner
```
## 2.2 注册 Runner
安装好 GitLab Runner 之后，我们只要启动 Runner 然后和 CI 绑定就可以了：
```
sudo gitlab-ci-multi-runner register
```
输入GitLab的URL：**http://192.168.122.95:85/ **
输入Token:     **jRsUbrby6txzMBpsY9tD**
输入 Runner 的名字: **my-gitlab-ci-runner**
输入 Runner 的标签: **my-gitlab-ci-runner-tag**
是否运行未标记的版本：**true**
是否将Runner锁定到当前项目:  **true**
输入Runner执行者：**shell**

当注册好 Runner 之后，可以用 sudo gitlab-ci-multi-runner list 命令来查看各个 Runner 的状态：
```
$ sudo gitlab-runner list
Listing configured runners                          ConfigFile=/etc/gitlab-runner/config.toml
my-gitlab-ci-runner                                 Executor=shell Token=P1EhC7SWzGVw9VxbYzWc URL=http://192.168.122.95:85/

```

# 3 添加.gitlab-ci.yml
```
stages:
    - build

GenerateHTML:
  stage: build
  script:
    - dir=`pwd`
    - echo $dir
    - gitbook build
```
# 4 安装GitBook
## 4.1 安装nodejs
首先需要安装 nodejs，以便能够使用 npm 来安装 gitbook。 从URL: https://nodejs.org/en/ 下载Linux版本的nodejs，我下载的是node-v8.11.4-linux-x64.tar.xz。
```
$ xz -d node-v8.11.4-linux-x64.tar.zx
$ tar xvf node-v8.11.4-linux-x64.tar
$ sudo cp node-v8.11.4-linux-x64 /opt/
$ sudo chmod 640 /etc/sudoers
$ sudo vi /etc/sudoers

Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/node-v8.11.4-linux-x64/bin/"

$ sudo chmod 440 /etc/sudoers
```
设置npm和gitbook命令的环境变量：
```
$ sudo vi /etc/profile

export NODEJS_HOME=/opt/node-v8.11.4-linux-x64
export PATH=${PATH}:${NODEJS_HOME}/bin
```

## 4.2 安装gitbook
```
$ sudo npm install -g gitbook-cli
```
## 4.3 添加文档库
实例文档库包含以下4个文件：
+ README.md
+ SUMMARY.md
+ .gitlab-ci.yml
+ your-doc.md

提交（git push） 后，会自动执行.gitlab-ci.yml中的脚本，build生成的网页在下面目录：
```
/home/gitlab-runner/builds/P1EhC7SW/0/gitlab_user/doc_repo/_book/
```

## 4.4 启动gitbook server
在GitLab服务器端，切换到gitlab-runner账户，安装完gitlab-runner后，默认不知道，所以需要先修改gitlab-runner用户密码。之后切换到gitlab-runner用户启动gitbook server.

```
$ sudo passwd gitlab-runner
$ su gitlab-runner
$ cd /home/gitlab-runner/builds/P1EhC7SW/0/gitlab_test/book_repo
$ gitbook serve &
```
浏览器访问：http://192.168.122.95:4000

