---
title: "记一次 写 CentOS 下 lnmp 一键编译脚本的经历"
date: 2019-01-30 22:04:00
slug: "lnmp-one-click-compilation-script-under-CentOS"
tags: ["shell", "LNMP"]
categories: [Dev]
draft: false
---

# 1. 引言
新购的云服务器需要配置 lnmp 环境，想着每次都重头手动编译来一次，太麻烦了效率低下，用别人的又未必符合自己的习惯跟风格，决定自己写一个一键安装的脚本。初出茅庐，欢迎指正，或建议或bug...后面考虑 做一个已经编译好的版本可供直接使用无需编译，以提升部署效率。
[脚本仓库](https://github.com/ethanzyang/centos-lnmp)
# 2. 自动脚本实现目标
* 编译 nginx 并初始化配置
* 编译 php 并初始化配置
* 编译 mysql 并初始化配置
* nginx 、php 需要配置 www-data 用户跟用户组
* 创建各自所需要的目录或者日志
* 配置 nginx/php/mysql 的 systemctl 开机自启，环境变量

# 3. 定义
**变量**
* start_dir 脚本所在目录
* install_dir 安装目录
* dl_dir 下载目录
* nginx_version nginx 版本
* php_version php 版本
* mysql_version mysql 版本
* mysql_data_dir mysql data 存放目录

**函数**
* init_yum
* init
* install_nginx
* install_php
* install_mysql
* print_conf

**目录结构**
```
install_dir-+---
            /php---+ #php配置文件
                   /php-fpm.conf.default #
                   /php.ini.default #
                   /www.conf.default #
            /nginx-+ #nginx 配置文件
                   /default.conf #
                   /vhost.conf #
            /pkg---+ #源码包
                   /boost_1_59_o.tar.gz
            install.sh #安装脚本
```
# 4. 安装
## 4.1 init
`init()` 初始化一些基础设置
`init_yum()` 安装所需要的依赖 如 wget 、gcc..
## 4.2 nginx
`install_nginx()` 安装 nginx
1. wget tar.gz 包
2. tar 解压
3. configure 参数并编编译 make & make install
4. 配置文件
4. 添加到 systemctl，添加环境变量
5. 开机自启 并 启动
6. 删除包跟源文件

## 4.3 php
`install_php()`
* 下载 并解压源码包
* configure 参数并编译
* 配置文件
* 添加到 systemctl，添加环境变量
* 开机自启 并 启动
* 删除包跟源文件

主意配置
配置文件 php-fpm.conf 会 include www.conf
脚本会 cp php/ 下面已经配置好的配置文件
```
# php-fpm.conf
pid = run/php-fpm.pid
www.conf
# listen 127.0.0.1:9000 or listen = 
user = www-data
group = www-data
listen = /usr/local/php7/var/run/run.sock
listen.owner = www-data
listen.group = www-data
```
## 4.4 mysql
mysql 编译需要 
* mysql5.7 及以上版本 需要 boost_1_59_0 及以上版本  [下载地址](https://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz)
* cmake，cmak 的一些参数跟选项的意义 暂时先看[这里](https://dev.mysql.com/doc/mysql-sourcebuild-excerpt/5.7/en/source-configuration-options.html)，得空了再整理

## 4.5 成功后启的进程
由 root 启 一个 master 进程，www-data 启 work 进程。

![file](https://cdn.learnku.com/uploads/images/201901/30/23174/2uYV2fUt0e.png!large)

# 5. 权限问题

* nginx 的日志目录 owner 跟 group 为 www-data
* nginx php-fpm work 进程都由 www-data 启
* php-fpm 的 unix socket 的用户跟组都为 www-data
* /data/www www根目录用户跟用户组为 www-data
* mysql 的 data 跟 log 目录owner group 都为 mysql
* mysql mariadb 的 owner 跟 group 为 mysql

使用 unix socket 注意

nginx 通过 `fast_cgi` 将请求给 `php-fpm`。
使用 unix socket 通信，需要启 nginx 的进程 与 php-fpm 的进程 对 unix socket 都要有 `rw` 权限
# 6. 写在最后
仓库地址
[https://github.com/jerr123/centos-lnmp](https://github.com/jerr123/centos-lnmp)

* shell 脚本还是很强大，里面很多指令不够熟悉。
* 对 php-fpm nginx mysql 以及 shell 都有了更深的认识。
* 多动手，多看日志，多看注释，多看官方文档，多看错误提示，多思考。