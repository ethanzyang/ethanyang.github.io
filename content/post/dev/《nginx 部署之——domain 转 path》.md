---
title: "《nginx 部署之——domain 转 path》"
date: 2020-05-23T08:15:05+08:00
slug: "nginx config about domain to path"
tags: [Nginx]
categories: [Dev]
draft: false
---

# 写在前面
时间精力总是有限的，由于把原本用来学习研究技术的大部分时间都用于了学习「how to be a trader」技术相关的已经很久没更新了！ 遗憾啦。
关于这个有兴趣或可以看[trading](https://github.com/jerr123/trading "trading")。
希望往后能把时间安排更好，技术也不要落下了！
# 引言
先说一个场景:
你有一个域名: abc.com，一台服务器ipx
1. 你有了你的第一个 web应用
```
你会把 abc.com ===> ipx，nginx 配置上解析
```
2. 你有了N个 web应用，但是你还是只有一个域名一台服务器
方案一
```
你可以用用多个域名来指向这台服务器
app1.abc.com ===> ipx
app2.abc.com ===> ipx
...
appN.abc.com ===> ipx
```
方案二：你可以为每个应用分配不同的端口
```
abc.com:80 ===> app1
abc.com:81 ===> app2
...
abc.com:N ===> appN
```
以上看上去都不是那么的「优雅」下面就是本篇文章的主角
方案三：domain转path
```
abc.com/app1 ===> app1
abc.com/app2 ===> app2
...
abc.com/appN ---> appN
```


# 如何配置？
配置中涉及的基础知识可以参考这里文中 [核心——你应该知道的基础知识](https://learnku.com/articles/44913#4d36d4) 部分

以「app1」为例

> 以下配置默认省略了`server`上下文
> 以下配置默认为`laravel`应用



## try_files

```sh
location ~* ^/app1(/(?<myPath>.*))? {
    alias /data/apps/app1/public/;
    try_files $uri $uri/ /index.php/app1/$myPath/?$query_string;
}
```

## fastcgi

```sh
location ~* ^(?<myScript>.+\.php)/app1(/(?<myPath>.*))?$ {
    alias /data/apps/app1/public/;
    fastcgi_split_path_info ^(.+\.php)/[a-zA-Z_\-]+(/.+)$;
    # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
    set $mySN /app1$myScript;

    fastcgi_pass 172.0.0.1:9000;
    fastcgi_index index.php;

    fastcgi_pass_request_headers on;

    include fastcgi_params;

    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $mySN;
    fastcgi_param PATH_INFO $fastcgi_path_info;
}
```

## 前端资源

```sh
location ~* ^/app1(?<myReal>/.+\.(js|css|png|jpg|jpeg|gif|ico|swf|eot|ttf|woff|woff2))$ {
    alias /data/apps/app1/public/;
    log_not_found off;
    access_log off;
    try_files $myReal $myReal/ 404;
}
```

# 核心——你应该知道的基础知识

你应该要知道
nginx 配置知识点
* location 语法规则
* nignx 正则相关
* fast_cgi

### location

```sh
# 基本语法
location [ = | ~ | ~ * | ^~ ] /uri/ { … }
```

#### [ = | ~ | ~ * | ^~ ] 部分

* `=` 精确匹配
* `~` 区别大小写
* `~*` 不区分大小写
* `^~` 

#### /uri/ 部分

简单的正则说明:

* `$` 以什么结尾
* `^` 以什么开头
* `*` 匹配任意字符
* `?` 匹配前面的子表达式零次或一次
* `.` 匹配除了换行符（\n）以外的任意一个字符
* `+` 表达式不出现或出现任意次
* `()`
    * 在被修饰匹配次数的时候，括号中的表达式可以作为整体被修饰
    * 取匹配结果的时候，括号中的表达式匹配到的内容可以被单独得到 


### try_files
```
try_files $uri $uri/ /index.php/app1/$myPath/?$query_string;

# get abc.com/app1/user/1?key=123
# $uri = /app1/user/1?key=123
```
- 先去找 $uri 指向的文件
- $uri/ 找 $uir/ 这个目录
- 最后去 `abc.com/index.php/app1/$myPath/?$query_string`

### alias
简单与 root 对比下，如果不明白就查资料吧
概念：
- alias    替换匹配部分的url
- root：替换整个url地址

e.g
配置：
location = `~* /app1`
alias = /data/app1/public/
root = /data/app1/public/

过程跟结果:

url: abc.com/app1/2.png
匹配部分: /app1 + /2.png
alias: /data/app1/public//2.png
root: /data/app1/public/app1/2.png

如果不知道正则写的对不对可以先校验
```php
$str = '/app1/user/1?key=123';
$isMatched = preg_match('/^\/app1(\/(?<myPath>.*))?/', $str, $matches);
print_r($matches);

/*
Array
(
    [0] => /app1/user/1?key=123
    [1] => /user/1?key=123
    [myPath] => user/1?key=123
    [2] => user/1?key=123
)
*/
```

### fastcgi

#### fastcgi_split_path_info
由于 nginx 默认获取不到 PATH_INFO 的值，
需要通过 fastcgi_split_path_info 指定定义的正则来捕获然后给 fastcgi_script_name $fastcgi_path_info 赋值。

`fastcgi_split_path_info ^(.+\.php)/[a-zA-Z_\-]+(/.+)$`

- $fastcgi_script_name = 第一个捕获的值
- $fastcgi_path_info = 第二个捕获的值


# 总结

基于知识点
* nginx 配置
* 正则表达式

对于不同的需求市场都会给出合理且成熟的解决方案，我们很多时候只是一个复读机。
如果你发现于你的需求市场并没有「优雅的方案」的时候，或许你就可以去试试。


> 以上正则表达式与nginx配置仅仅列举出了本篇文章中用到的部分重点知识点
> 系统性的正则表达式知识可以参考 [揭开正则表达式的神秘面纱](http://www.regexlab.com/zh/regref.htm)
> 系统性的nginx配置就自己查查吧（没有好的文档推荐）有空整理一份完整的配置参考

