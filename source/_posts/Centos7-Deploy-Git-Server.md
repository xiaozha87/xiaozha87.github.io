---
title: Centos7搭建Git服务器
date: 2018-08-25 09:23:15
tags:
- Github
category: Github
---

环境说明
- CentOS 7.x 最小安装
- 配置网络连接

# 1. 安装Git及创建用户
``` bash
# 安装Git
$ yum install git
$ git --version

# 创建一个git用户组和用户，用来运行git服务
$ groupadd git
$ adduser git -g git
```
<!-- more -->
禁止git用户登录,修改/etc/passwd文件

``` bash
# 找到这句：
git:x:503:503::/home/git:/bin/bash

# 改为：
git:x:503:503::/home/git:/bin/git-shell
```
# 2. 创建证书登录

``` bash
$ mkdir /home/git/.ssh
$ chmod 700 /home/git/.ssh
$ touch 700 /home/git/.ssh/authorized_keys
$ chmod 600 /home/git/.ssh/authorized_keys
```
注意，如果是采用的sudo方式来创建git和相应的文件的，需要设置/home/git/.ssh/的owner为git，否则还是每次要输入密码的。
``` bash
# owner改为git
$ sudo chown -R git:git /home/git/.ssh/
```
编辑`/home/git/.ssh/authorized_keys`，把客户端的公钥id_rsa.pub放进去，1个公钥1行。
``` bash
# 创建私钥，文件位于用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件
$ ssh-keygen -t rsa -C "youremail@example.com"
```
在服务器端打开RSA认证,在文件/etc/ssh/sshd_config中添加或者修改下列三行内容:
``` bash
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile  .ssh/authorized_keys
```
# 3.初始化Git仓库
```bash
$ cd /srv
$ mkdir gitrepo
$ chown git:git gitrepo/
$ cd gitrepo //切换到希望创建工作区的目录

# 创建一个空的Git仓库，服务器上的Git仓库通常都以.git结尾,会创建一个唯一分支master
$ git init --bare project.git

# 将仓库所属用户改为git
$ chown -R git:git project.git
```
# 4. 克隆仓库
在另一台电脑上（下面称为：Client），安装git，并且运行下面的命令：
```bash
$ git clone git@10.10.10.10:/srv/gitrepo/project.git
```
# 5. 验证安装
## 5.1. 推送到远程服务器
在本地Client的project目录下，创建一个文件：text.txt，内容随意，然后上传到远端：
```bash
git push origin master
```
## 5.2. 检验
在本地Client的另外一个目录下，克隆一下：
`$ git clone git@10.10.10.10:/srv/gitrepo/project.git`
看看text.txt文件是否存在，内容是否对。
## 5.3. 常用的Git命令：
```bash
$ git add . 添加所有文件  注意有个 .
$ git commit -m '注释' 提交本地
$ git push origin master提交给默认分支
$ git -rm 删除
$ git pull origin master 从默认分支下载
$ git branch -v 查看所有分支
```
# 6. 设置git钩子
进入我们的裸仓库的hooks文件夹，然后新建一个post-receive文件。
```bash
$ cd hooks/
$ vim post-receive

# 在post-receive写入以下内容：

#!/bin/bash
git --work-tree=/www/laravel checkout -f
```
/www/laravel为实际需要同步的站点目录。
然后修改post-receive为可执行文件（其实这就是一个脚本文件）

`chmod +x post-receive`

post-receive的原理就是，当远程仓库发现有用户执行了push操作，就会执行一个脚本post-receive（钩子）。

注意：同时还需要修改web站点目录的权限，修改所属用户与用户组为git，否则钩子的权限可能会不足而导致执行失败。（也可以通过添加git用户到相应的用户组来解决问题）
`chown git:git -R /www    # 修改所属用户`
设置好钩子后，当你本地再次执行push的时候，发现web目录的文件也同步的更新了。