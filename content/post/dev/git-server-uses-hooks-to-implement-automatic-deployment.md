---
title: "Git 服务器 利用 hook 实现自动部署"
date: 2018-12-10 11:53:00
slug: "git-server-uses-hooks-to-implement-automatic-deployment"
tags: ["git", "automatic-deployment"]
categories: [Dev]
draft: false
---

# git 自动部署
## 原理
1. 客户端 `push` 之后会触发 `git hook` 执行 `hook`下面的 `post-receive`
2. 通过 `post-receive` 执行`shell`脚本将在 web 目录下拉取项目
## 实现
s 表示 git 服务器端，c 表示 git 客户端
### 搭建 git 服务器
#### 添加 git 账号
```shell
$ groupadd git;
$ useradd -d /home/git -m git
```
#### 创建证书 配置 git 账户 ssh
ssh 配置文件在 /etc/ssh/sshd_config里面,修改里面内容
```
RSAAuthentication yes   
PubkeyAuthentication yes   
AuthorizedKeysFile .ssh/authorized_keys
```
创建 git 的 .ssh
将允许进行登录的公钥放在 `authorized_keys` 里面一个一行
如果失败,尝试执行`ssh-keygen`重新生成 RSA
```
cd /home/git/
mkdir .ssh
chmod 755 .ssh
touch .ssh/authorized_keys
chmod 644 .ssh/authorized_keys
```
#### 初始化 Git 仓库
```shell
cd /home/git
git init --bare uucheers.git
chown -R git:git uucheers.git
#克隆
git clone git@ip:uucheers.git
git clone git@ip:/home/git/uucheers.git
#在项目目录里面 git clone 过去 
git clone /home/git/uucheers.git
#同时 更改权限 不然 git 用户没法写入
```

###  hook
 在项目目录下
```shell
cd /home/git/uucheers/hooks
vim post-receive
```
输入如下内容
```shell
#!/bin/sh
#
#判断是不是远端仓库
IS_BARE=$(git rev-parse --is-bare-repository)
if [ -z "$IS_BARE" ]; then
echo >&2 "fatal: post-receive: IS_NOT_BARE $IS_BARE"
exit 1
fi

unset GIT_DIR
DeployPath="/usr/local/src/uucheers-docker/app/uucheers"
echo "==============================================="
cd $DeployPath
echo "deploying the project"
#git stash
#git pull origin master
git fetch --all
git reset --hard origin/master
time=`date`
echo "web server pull at webserver at time: $time."
echo "================================================"
```
`chmod +x post-receive` 添加执行权限
