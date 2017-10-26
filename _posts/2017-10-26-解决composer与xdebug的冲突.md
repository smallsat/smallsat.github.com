---
layout: post
title: '解决composer与xdebug的冲突'
date: 2017-10-26
author: smallsat
categories: composer xdebug
tags: composer php xdebug
---
安装了 `xudebug` 后使用 `composer` 会产生一个警告,网上的资料说主要是对 `composer` 的性能有较大影响。
下面介绍一下如何解决。
1. 查看php配置文件位置
```shell
    php-i|grep "php.ini"
```
可以获取配置文件所在的目录
``` shell
Configuration File (php.ini) Path =>  /www/server/php/70/etc/
Loaded Configuration File => /www/server/php/70/etc/php.ini
```
2. 进入文件所在位置，复制一份新文件，并将新文件中的 `xdebug` 关闭
3. 创建命令别名
*注意：先查看`composer`所在的目录*
```shell
alias composer="php -c /www/server/php/70/etc/no-xdebug-php.ini /usr/bin/composer"
```
