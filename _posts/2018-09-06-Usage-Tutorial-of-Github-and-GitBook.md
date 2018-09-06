# 1. GitHub
## 1.1. 创建账户
https://github.com/  在上面注册
username: aojunhu
## 1.2. 生成本地公钥/私钥对
在本地Ubuntu上，ssh-keygen命令创建公钥/私钥对：
```
$ ssh-keygen
```
然后根据提示在用户主目录下的.ssh目录中创建默认的公钥/私钥对文件，其中~/.ssh/id_rsa是私钥文件，~/.ssh/id_rsa.pub是公钥文件。注意私钥文件要严加保护，不能泄露给任何人。如果在执行ssh-keygen命令时选择了使用口令保护私钥，私钥文件是经过加密的。至于公钥文件~/.ssh/id_rsa.pub则可以放心地公开给他人。
也可以用ssh-keygen命令以不同的名称创建多个公钥，当拥有多个GitHub账号时，非常重要。这是因为虽然一个GitHub账号允许使用多个不同的SSH公钥，但反过来，一个SSH公钥只能对应于一个GitHub账号。下面的命令在~/.ssh目录下创建名为aojunhu的私钥和名为aojunhu.pub的公钥文件。
```
$ ssh-keygen -C "gotgithub@gmail.com" -f ~/.ssh/aojunhu

$ ls ~/.ssh/
aojunhu  aojunhu.pub
```
当生成的公钥/私钥对不在缺省位置（~/.ssh/id_rsa等）时，使用ssh命令连接远程主机时需要使用参数-i <filename>指定公钥/私钥对。或者在配置文件~/.ssh/config中针对相应主机进行设定。例如对于上例创建了非缺省公钥/私钥对~/.ssh/aojunhu，可以在~/.ssh/config配置文件中写入如下配置。
```
Host github.com
    User git 
    Hostname github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/aojunhu               
```
好了，有了上面的准备，就将~/.ssh/aojunhu.pub文件内容拷贝到剪切板。然后将公钥文件中的内容粘贴到GitHub的SSH公钥管理的对话框中。
设置成功后，再用ssh命令访问GitHub，会显示一条认证成功信息并退出。在认证成功的信息中还会显示该公钥对应的用户名。
```
$ ssh -T git@github.com
Hi aojunhu! You've successfully authenticated, but GitHub does not provide shell access.
```

## 1.3. 项目托管
本章介绍如何在GitHub上创建一个新项目，包括创建版本库及为项目设计主页等。
### 1.3.1. 创建新项目
Your repositories --> Create a new repository  --> set Repository name --> Create Reopsitory Button
### 1.3.2. 版本库初始化
如果是从头创建版本库，可以采用先克隆，建立提交数据，最后再通过推送完成GitHub版本库的初始化。步骤如下：
+ 克隆版本库。
```
$ git clone git@github.com:aojunhu/helloworld.git helloworld
Cloning into 'helloworld'...
Warning: Permanently added the RSA host key for IP address '13.250.177.223' to the list of known hosts.
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
Checking connectivity... done.
```
### 1.3.3. 创建并修改文件README.md
```
# My first GitHub project
This is a example repository.

## Support markdown syntax format.
example:
这是项目 [helloworld](https://github.com/aojunhu/helloworld)，欢迎访问。
* item1
* item2

```
### 1.3.4. 添加README.md文件并提交。
```
$ git add README.md
$ git commit -m "README for this project."
```
### 1.3.5. 向GitHub推送，完成版本库初始化。
```
$ git push origin master
```
### 1.3.6. 从已有版本库创建
如果在GitHub项目初始化之前，数据已经存在于本地版本库中，显然像上面那样先克隆、再提交、后推送的方法就不适宜了。应该采用下面的方法。
为试验新的版本库初始化方法，先把刚刚新建的测试项目“helloworld”删除，同时也将本地工作区中克隆的“helloworld”删除。警告：删除项目的操作非常危险，不可恢复，慎用。
* 点击项目首页中"Settings"选项卡，在"Danger Zone"区域有"Delete this repository"
* 然后再重建版本库“helloworld”，见3.1节。
* 本地建立一个Git版本库。
```
$ mkdir helloworld
$ cd helloworld
$ git init
```
* 然后在版本库中添加示例文件，如README.md文件，内容同前。
```
$ git add README.md
$ git commit -m "README for this project."
```
* 为版本库添加名为origin的远程版本库。
```
$ git remote add origin git@github.com:aojunhu/helloworld.git
```
* 执行推送命令，完成GitHub版本库的初始化。注意命令行中的-u参数，在推送成功后自动建立本地分支与远程版本库分支的追踪。
```
$ git push -u origin master
```
## 1.4. 操作版本库
### 1.4.1. 强制推送
上文提交时，提交用户设置的邮件地址并非gotgithub用户设置的邮件地址。补救办法就是对此提交进行修改，然后强制推送到GitHub。
* 重新设置user.name和user.email配置变量。
```
$ git config user.name "naiquan.hu"
$ git config user.email "naiquan_hu@hotmail.com"
```
* 执行Git修补提交命令。
注意使用参数--reset-author会将提交信息中的属性Author连同AuthorDate一并修改，否则只修改Commit和CommitDate。参数-C HEAD维持提交说明不变。
```
$ git commit --amend --reset-author -C HEAD
```
* 查看提交日志，发现提交者信息和作者信息都已经更改。
```
$ git log --pretty=fuller
commit 959af4fe61bbdc984dcc26acd139ea3e164096f4
Author:     naiquan.hu <naiquan_hu@hotmail.com>
AuthorDate: Tue Sep 4 15:58:25 2018 +0800
Commit:     naiquan.hu <naiquan_hu@hotmail.com>
CommitDate: Tue Sep 4 15:58:25 2018 +0800

    README for this project.
```
* 使用强制推送。
```
$ git push -f
```
### 1.4.2. 新建分支
Git的分支就是保存在.git/refs/heads/命名空间下的引用。引用文件的内容是该分支对应的提交ID。当前版本库中的默认分支master就对应于文件.git/refs/heads/master。
若在GitHub版本库中创建分支，首先要在本地版本库中创建新的分支（即引用），然后用推送命令将本地创建的新的引用连同所指向的提交推送到GitHub版本库中完成GitHub上分支的创建。操作如下：
* 本地版本库中建立新分支mybranch1。
创建分支有多种方法，如使用git branch命令，但最为便捷的就是git checkout -b命令，同时完成新分支的创建和分支切换。
```
$ git checkout -b mybranch1
Switched to a new branch 'mybranch1'
```
* 为了易于识别，添加一个新文件hello1，并提交。
```
$ touch hello1
$ echo "How are you!" > hello1 
$ git add hello1 
$ git commit -m "add hello1."
[mybranch1 f65bdb4] add hello1.
 1 file changed, 1 insertion(+)
 create mode 100644 hello1
```
* 通过推送命令，将本地分支mybranch1推送到GitHub远程版本库，完成在GitHub上的新分支创建。
```
$ git push -u origin mybranch1
```
在GitHub上查看版本库，会看到新增了一个分支mybranch1，不过默认分支仍为master。
### 1.4.3. 设置默认分支
在"Settings"选项卡可以设置Default branch。
修改了GitHub默认分支后，如果再从GitHub克隆版本库，本地克隆后版本库的默认分支也将改变。
```
$ git clone git@github.com:aojunhu/helloworld.git helloworld-branch1Cloning into 'helloworld-branch1'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
Checking connectivity... done.
$ cd helloworld-branch1/
$ git branch
* mybranch1
```
实际上修改GitHub上版本库的默认分支，就是将GitHub版本库的头指针HEAD指向了其他分支，如mybranch1分支。这可以从下面命令看出。
```
$ git branch -r
  origin/HEAD -> origin/mybranch1
  origin/master
  origin/mybranch1
$ cd helloworld-master/
$ git branch -r
  origin/HEAD -> origin/master
  origin/master
  origin/mybranch1
```
### 1.4.4. 删除分支
不能删除当前工作分支。因此先切换到其他分支，例如从GitHub版本库中取出master分支并切换。
```
$ git branch
  master
* mybranch1
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ git branch
* master
  mybranch1
```
现在可以删除mybanch1分支。下面的命令之所以使用-D参数，而非-d参数，是因为Git在删除分支时为避免数据丢失，默认禁止删除尚未合并的分支。参数-D则可强制删除尚未合并的分支。
```
$ git branch -D mybranch1
Deleted branch mybranch1 (was f46a284).
```
现在只是本地分支被删除了，远程GitHub服务器上的mybranch1分支尚在。删除远程GitHub版本库中的分支就不能使用git branch命令，而是要使用git push命令，而且要先在GitHub上把默认分支切换到mater。在使用推送分支命令时要使用一个特殊的引用表达式（冒号前为空）。
```
$ git push origin :mybranch1
```
### 1.4.5. 里程碑管理
里程碑即tag，其管理和分支管理非常类似。里程碑和分支一样也是以引用的形式存在的，保存在.git/refs/tags/路径下。引用可能指向一个提交，但也可能是其他类型（Tag对象）。
* 轻量级里程碑：用git tag <tagname> [<commit>] 命令创建，引用直接指向一个提交对象<commit>。
* 带说明的里程碑：用git tag -a <tagname> [<commit>] 命令创建，并且在创建时需要提供创建里程碑的说明。Git会创建一个tag对象保存里程碑说明、里程碑的指向、创建里程碑的用户等信息，里程碑引用指向该Tag对象。
* 带签名的里程碑：用git tag -s <tagname> [<commit>] 命令创建。是在带说明的里程碑的基础上引入了PGP签名，保证了所创建的里程碑的完整性和不可拒绝性。
下面演示一下里程碑的创建和管理。
* 先在本地创建一个新提交。
```
$ touch hello1
$ git add hello1
$ git commit -m "add hello1 for mark."
```
* 本地创建里程碑mytag1、mytag2和mytag3。
```
$ git tag -m "Tag on initial commit" mytag1 HEAD^
$ git tag -m "Tag on new commit"     mytag2
$ git tag mytag3
```
* 查看新建立的里程碑。
```
$ git tag -l -n1
mytag1          Tag on initial commit
mytag2          Tag on new commit
mytag3          add hello1 for mark.
```
* 将本地里程碑推送到GitHub远程版本库。
```
$ git push origin refs/tags/*
Counting objects: 6, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 548 bytes, done.
Total 5 (delta 0), reused 0 (delta 0)
To git@github.com:gotgithub/helloworld.git
 * [new tag]         mytag1 -> mytag1
 * [new tag]         mytag2 -> mytag2
 * [new tag]         mytag3 -> mytag3
```
* 删除本地里程碑。
```
$ git tag -d mytag3
Deleted tag 'mytag3' (was c71231c)
```
* 删除GitHub远程版本库中的里程碑。
```
$ git push origin :mytag3
To git@github.com:gotgithub/helloworld.git
  [deleted]         mytag3
```

另外，版本库还可以与一些扩展应用建立联系，比如跟Email 和Redmine整合，Email可以实现新提交推送至版本库时，发送通知邮件。Redmine可以和多种版本库（包括Git）整合，可以直接通过Web界面浏览Git提交，还实现了提交和问题的关联。
## 1.5. 建立主页
### 1.5.1. 创建个人主页
GitHub 为每一个用户分配了一个二级域名<user-id>.github.io，用户为自己的二级域名创建主页很容易，只要在托管空间下创建一个名为<user-id>.github.io的版本库，向其master分支提交网站静态页面即可，其中网站首页为index.html。下面以aojunhu用户为例介绍如何创建个人主页。
* 用户aojunhu创建一个名为aojunhu.github.io的Git版本库。
* 在本地克隆新建立的版本库。
```
$ git clone git@github.com:aojunhu/aojunhu.github.io.git aojunhu.github.io
Cloning into 'aojunhu.github.io'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
Checking connectivity... done.
$ cd aojunhu.github.io/
```
* 在版本库根目录中创建文件index.html作为首页。
```
$ printf "<h1>GotGitHub's HomePage</h1>It works.\n" > index.html
```
* 提交
```
$ git add index.html
$ git commit -m "Homepage test version."
```
* 推送到GitHub，完成远程版本库创建。
```
$ git push origin master
```
* 访问网址： [http://aojunhu.github.io/](http://aojunhu.github.io/) 。

### 1.5.2. 创建项目主页
如前所述，GitHub会为每个账号分配一个二级域名<user-id>.github.io作为用户的首页地址。实际上还可以为每个项目设置主页，项目主页也通过此二级域名进行访问。

例如gotgithub用户创建的helloworld项目如果启用了项目主页，则可通过网址http://gotgithub.github.io/helloworld/访问。

为项目启用项目主页很简单，只需要在项目版本库中创建一个名为gh-pages的分支，并向其中添加静态网页即可。也就是说如果项目的Git版本库中包含了名为gh-pages分支的话，则表明该项目提供静态网页构成的主页，可以通过网址http://<user-id>.github.io/<project-name>访问到。

下面以用户aojunhu的项目helloworld为例，介绍如何维护项目主页。

如果本地尚未从GitHub克隆helloworld版本库，执行如下命令。
```
$ git clone git@github.com:aojunhu/helloworld.git
$ cd helloworld
```
当前版本库只有一个名为master的分支，如果直接从master分支创建gh-pages分支操作非常简单，但是作为保存网页的gh-pages分支中的内容和master分支中的可能完全不同。如果不希望gh-pages分支继承master分支的历史和文件，即想要创建一个干净的gh-pages分支，需要一点小技巧。

若使用命令行创建干净的gh-pages分支，可以从下面三个方法任选一种。

第一种方法用到两个Git底层命令：git write-tree和git commit-tree。步骤如下：
* 基于master分支建立分支gh-pages。
```
$ git checkout -b gh-pages
```
* 删除暂存区文件，即相当于清空暂存区。
```
$ rm .git/index
```
* 创建项目首页index.html。
```
$ printf "hello world.\n" > index.html
```
* 添加文件index.html到暂存区。
```
$ git add index.html
```
* 用Git底层命令创建新的根提交，并将分支gh-pages重置。
```
$ git reset --hard $(echo "branch gh-pages init." | git commit-tree $(git write-tree))
```
* 执行推送命令，在GitHub远程版本库创建分支gh-pages。
```
$ git push -u origin gh-pages
```
稍后，用浏览器访问下面的地址即可看到刚刚提交的项目首页： [http://aojunhu.github.io/helloworld/](http://aojunhu.github.io/helloworld/) 。
当然，还有其他创建方法，详见：[GotGitHub](http://www.worldhello.net/gotgithub/03-project-hosting/050-homepage.html)

# 2. GitBook
本文以Ubuntu 14.04.5 LTS为例安装。
## 2.1. 安装nodejs
首先需要安装 nodejs，以便能够使用 npm 来安装 gitbook。
从URL: [https://nodejs.org/en/](https://nodejs.org/en/)下载Linux版本的nodejs，我下载的是node-v8.11.4-linux-x64.tar.xz。
```
$ xz -d node-v8.11.4-linux-x64.tar.zx
$ tar xvf node-v8.11.4-linux-x64.tar
$ sudo cp node-v8.11.4-linux-x64 /opt/
$ sudo chmod 640 /etc/sudoers
$ sudo vi /etc/sudoers

Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/node-v8.11.4-linux-x64/bin/"

$ sudo chmod 440 /etc/sudoers
```
重启系统。
## 2.2. 安装gitbook
```
$ sudo npm install -g gitbook-cli
```
## 2.3. 基本使用
gitbook 的基本用法非常简单，基本上就只有两步：

使用 gitbook init 初始化书籍目录
使用 gitbook serve 编译书籍
下面将结合一个非常简单的实例，来介绍 gitbook 的基本用法。
### 2.3.1. gitbook init
首先，创建如下目录结构：
```
$ tree book/
book/
├── README.md
└── SUMMARY.md

0 directories, 2 files
```
README.md 和 SUMMARY.md 是两个必须文件，README.md 是对书籍的简单介绍：
```
$ cat book/README.md 
# README

This is a book powered by [GitBook](https://github.com/GitbookIO/gitbook).
```
SUMMARY.md 是书籍的目录结构。内容如下：
```
$ cat book/SUMMARY.md 
# SUMMARY

* [Chapter1](chapter1/README.md)
  * [Section1.1](chapter1/section1.1.md)
  * [Section1.2](chapter1/section1.2.md)
* [Chapter2](chapter2/README.md)
```
创建了这两个文件后，使用 gitbook init，它会为我们创建 SUMMARY.md 中的目录结构。
```
$ cd book
$ gitbook init
$ tree
.
├── chapter1
│   ├── README.md
│   ├── section1.1.md
│   └── section1.2.md
├── chapter2
│   └── README.md
├── README.md
└── SUMMARY.md

2 directories, 6 files
```
注意：在我的实验中，gitbook init 只支持两级目录！
### 2.3.2. gitbook serve
书籍目录结构创建完成以后，就可以使用 gitbook serve 来编译和预览书籍了：
```
$ gitbook serve
Live reload server started on port: 35729
Press CTRL+C to quit ...

info: 7 plugins are installed 
info: loading plugin "livereload"... OK 
info: loading plugin "highlight"... OK 
info: loading plugin "search"... OK 
info: loading plugin "lunr"... OK 
info: loading plugin "sharing"... OK 
info: loading plugin "fontsettings"... OK 
info: loading plugin "theme-default"... OK 
info: found 5 pages 
info: found 0 asset files 
info: >> generation finished with success in 1.1s ! 

Starting server ...
Serving book on http://localhost:4000
```
`gitbook serve` 命令实际上会首先调用 `gitbook build` 编译书籍，完成以后会打开一个 web 服务器，监听在本地的 4000 端口。
现在，可以用浏览器打开 [http://127.0.0.1:4000](http://127.0.0.1:4000/) 查看书籍的效果。
现在，gitbook 为我们创建了书籍目录结构后，就可以向其中添加真正的内容了，文件的编写使用 markdown 语法，在文件修改过程中，每一次保存文件，`gitbook serve` 都会自动重新编译，所以可以持续通过浏览器来查看最新的书籍效果！
另外，用户还可以下载 [gitbook 编辑器](https://github.com/GitbookIO/editor)，做到所见即所得的编辑

# 3. GitHub + GitBook管理图书
## 3.1. 书籍内容存储
使用github提供的git仓库管理书籍内容，增加、修改书籍内容，提交至git仓库即可。
+ github上创建仓库arm-cm4-guide
+ checkout arm-cm4-guide到本地
```
$ git clone git@github.com:aojunhu/arm-cm4-guide.git arm-cm4-guide
```
+ 编写书籍内容
```
$ cd arm-cm4-guide/
$ cat README.md 
# arm-cm4-guide

ARM Cortex-M3与Cortex-M4权威指南（第三版）

$ cat SUMMARY.md 
# SUMMARY

* [ARM Cortex-M 处理器简介](chapter1/README.md)
    * [什么是ARM Cortex-M处理器](chapter1/section1.md)
        * [Cortex-M处理器家族](chapter1/section1.1.md)
        * [处理器和微控制器的区别](chapter1/section1.2.md)
    * [ARM Cortex-M处理器的优势](chapter1/section2.md)
        * [低功耗](chapter2/section2.1.md)
        * [代码密度](chapter2/section2.2.md)
$ tree
.
├── README.md
└── SUMMARY.md

0 directories, 2 files

$ gitbook init
info: create chapter1/README.md 
info: create chapter1/section1.md 
info: create chapter1/section1.1.md 
info: create chapter1/section1.2.md 
info: create chapter1/section2.md 
info: create chapter2/section2.1.md 
info: create chapter2/section2.2.md 
info: create SUMMARY.md 
info: initialization is finished

$ tree
.
├── chapter1
│   ├── README.md
│   ├── section1.1.md
│   ├── section1.2.md
│   ├── section1.md
│   └── section2.md
├── chapter2
│   ├── section2.1.md
│   └── section2.2.md
├── README.md
└── SUMMARY.md

2 directories, 9 files
```
+ 提交到arm-cm4-guide仓库
```
$ git add README.md SUMMARY.md chapter1/ chapter2/
$ git commit -m "add book content."
$ git push origin master
```
## 3.2. 构建书籍
首先，使用 gitbook build 将书籍内容输出到默认目录，也就是当前目录下的 _book 目录。
```
$ gitbook build
info: 7 plugins are installed 
info: 6 explicitly listed 
info: loading plugin "highlight"... OK 
info: loading plugin "search"... OK 
info: loading plugin "lunr"... OK 
info: loading plugin "sharing"... OK 
info: loading plugin "fontsettings"... OK 
info: loading plugin "theme-default"... OK 
info: found 8 pages 
info: found 0 asset files 
info: >> generation finished with success in 1.5s ! 

$ ls _book/
chapter1  chapter2  gitbook  index.html  search_index.json
```
## 3.3. 创建 gh-pages 分支
将gitbook生成的html page发布github pages上(实际是提交内容到gh-pages 分支)。所以先创建arm-cm4-guide的 gh-pages 分支。
```
$ git checkout -b gh-pages
$ rm .git/index
$ git reset --hard $(echo "branch gh-pages init." | git commit-tree $(git write-tree))
$ git push -u origin gh-pages
```
现在，目录下应该只剩下 _book 目录了，首先，忽略一些文件：
```
$ echo "*~" > .gitignore
$ echo "_book" >> .gitignore
$ git add .gitignore
$ git commit -m "Ignore some files"
```
然后，加入 _book 下的内容到分支中：
```
$ cp -r _book/* .
$ git add .
$ git commit -m "Publish book"
```
## 3.4. 上传书籍内容到 GitHub
现在，可以将编译好的书籍内容上传到 GitHub 中 arm-cm4-guide项目的 gh-pages 分支了.
```
$ git push -u origin gh-pages
```
访问 https://aojunhu.github.io/arm-cm4-guide/ 就可以阅读 arm-cm4-guide这本书了！

# 4. 利用 GitHub Pages 快速搭建个人博客
fork别人的模板（qiubaiying.github.io），rename为自己的仓库名(<user-id>.github.io)。