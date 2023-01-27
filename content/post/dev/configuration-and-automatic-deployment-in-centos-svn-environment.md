---
title: " CentOS SVN 环境搭建配置跟自动部署"
date: 2018-08-03 10:19:50
slug: "configuration-and-automatic-deployment-in-centos-svn-environment"
tags: ["svn", "automatic-deployment"]
categories: [Dev]
draft: false
---

## 一. 安装subversion并创建版本库

### 1.安装subversion
```shell
yum -y install  subversion
```
安装成功后查看版本号：
`svnserve --version`

### 2. 创建版本库

a.创建目录
`mkdir /var/svn`

b.创建版本库
`svnadmin create /var/svn/repository`

c.查看创建情况
`cd /var/svn/repository`
`ll`
## 二。配置基础信息
注意：所有的配置项都需要顶格，即前面不能预留空格，否则报错

`cd /var/svn/readerstar/conf`

### 1.配置SVN服务综合配置文件svnserve.conf

`vim  svnserve.conf`

配置以下内容： 
```sh
anon-access = read/none #匿名用户可读/不可读
auth-access = write #授权用户可写
password-db = passwd #使用哪个文件作为账号文件
authz-db = authz #使用哪个文件作为权限文件
realm = /home/svn/repository #认证空间名，版本库所在目录
```
### 2.配置用户组
` vim authz`
 ```sh
 [groups]

admin = hongcoo,hello

#admin用户组 hongcoo 用户

[weixin:/]
@admin = rw

#用户组admin对repository库有读写权限
```
### 3.配置用户名密码
    
    `vim passwd`
```sh
[users]
hongcoo = hongcoo
```
### 4.启动svn
`svnserve -d -r /var/svn/repository`


检查服务是否启动成功
`ps aux | grep svn`

通过netstat可以看到SVN打开了3690端口
`netstat -tnlp`

设置成开机启动
`systemctl enable svnserve.service`



### 5.测试项目情况
`svn co svn://localhost/repository`



## 三。配置svn更新自动同步到web目录

### 1.先执行checkout
 `svn co svn://localhost/weiqing /home/www/repository --username libin --password libin123`
 
 ### 2.建立post-commit文件
 `cd /var/svn/repository/hooks`
 
 `cp /var/svn/readerstar/hooks/post-commit.tmpl  /var/svn/readerstar/hooks/post-commit`
 
 `vim /var/svn/readerstar/hooks/post-commit`
 
 
 配置内容：
 ```sh
REPOS="$1"
REV="$2"

export LANG=zh_CN.UTF-8

SVN_PATH=/usr/bin/svn
WEB_PATH=/home/www/weixin.com
LOG_PATH=/tmp/svn_update.log

$SVN_PATH update $WEB_PATH  --username libin  --password libin123  --no-auth-cache
```
  修改post-commit用户为www目录用户
  `chown www:www post-commit `
  
  `chmod +x post-commit`