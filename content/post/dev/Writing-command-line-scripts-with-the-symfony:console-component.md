---
title: "用 symfony/console 组件写命令行脚本"
date: 2019-01-21 22:03:00
slug: "Writing-command-line-scripts-with-the-symfony/console-component"
tags: ["PHP", "symfony-console"]
categories: [Dev]
draft: false
---

# 前言
`php artisan model:create User` 这条命令你一定很熟悉。
下面我们就来实现类似的命令。
# symfony/console 是什么
首先要明白 `symfony/console` 是什么?
它是 symfony 里面的一个控制台命令组件，更优秀的事 symfony 的组件各自都保持独立，不需要其他依赖。这就意味着我们可以在任意我们想要的地方去使用。
# 如何编写 console 脚本
1. composer 安装 symfony/console 组件。
2. 按照规范编写 console 应用程序（等于 artisan ）。
3. 按照规范编写 commands （命令）。
4. 大功告成。

# 安装
`composer require symfony/console`
# 编写 console 程序
`console_command`文件
```php
#!/usr/bin/env php
<?php

require __DIR__.'/vendor/autoload.php';

use Symfony\Component\Console\Application;
use Cmd\ModelCommand;
 
$application = new Application();
 
// 注册我们编写的命令 (commands)
$application->add(new ModelCommand());
 
$application->run();
```
# 编写 command 程序
这里需要注意自动加载问题！
 ```
 "autoload": {
       "psr-4":{
           "Cmd\\": "Cmd"
       }
 ```
 上面一段加入到 `composer.json` 中。下面是我的最终文件内容
```
{
    "require": {
        "symfony/console": "^4.2"
    },
    "autoload": {
       "psr-4":{
           "Cmd\\": "Cmd"
       }
   }
}
```
`ModelCommand.php`
```php
<?php
namespace Cmd;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputArgument;

class ModelCommand extends Command
{
    protected function configure()
    {
        $this
            // 命令的名称 （"php console_command" 后面的部分）
            ->setName('model:create')
            // 运行 "php console_command list" 时的简短描述
            ->setDescription('Create new model')
            // 运行命令时使用 "--help" 选项时的完整命令描述
            ->setHelp('This command allow you to create models...')
            // 配置一个参数
            ->addArgument('name', InputArgument::REQUIRED, 'what\'s model you want to create ?')
            // 配置一个可选参数
            ->addArgument('optional_argument', InputArgument::OPTIONAL, 'this is a optional argument');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // 你想要做的任何操作
        $optional_argument = $input->getArgument('optional_argument');

        $output->writeln('creating...');
        $output->writeln('created ' . $input->getArgument('name') . ' model success !');
        
        if ($optional_argument)
            $output->writeln('optional argument is ' . $optional_argument);

        $output->writeln('the end.');
    }
}
```
# 大功告成
![file](https://cdn.learnku.com/uploads/images/201901/21/23174/4ecQJS58AR.png!large)