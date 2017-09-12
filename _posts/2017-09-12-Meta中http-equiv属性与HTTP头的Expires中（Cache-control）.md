---
layout: post
title: 'Meta中http-equiv属性与HTTP头的Expires中（Cache-control）'
date: 2017-09-12
author: smallsat
categories: html cache
tags: html meta cache
---



## 一、概述

1. `http-equiv` 顾名思义，相当于 `http` 的文件头作用，它可以向浏览器传回一些有用的信息，以帮助正确和精确地显示网页内容，与之对应的属性值为`content` ，`content` 中的内容其实就是各个参数的变量值。

2. `Cache-control`用于控制`HTTP` 缓存（在`HTTP/1.0` 中可能部分没实现，仅仅实现了`Pragma: no-cache）`

## 二、基本内容

`meat` 标签的`http-equiv`属性语法格式是：
```html
<meta http-equiv="参数" content="参数变量值">
```
其中`http-equiv` 属性主要有以下几种参数
1. `Expires`
说明：可以用于设定网页的到期时间。一旦网页过期，必须到服务器上重新传输。 
用法：
```html
<meta http-equiv="expires" content="Wed, 20 Jun 2007 22:33:00 GMT">
```
**注意：必须使用`GMT`的时间格式。**
2. `Pragma`(`cache`模式)
说明：是用于设定禁止浏览器从本地机的缓存中调阅页面内容，设定后一旦离开网页就无法从`Cache`中再调出。
用法：
```html
<meta http-equiv="Pragma" content="no-cache">
```
**注意：这样设定，访问者将无法脱机浏览。**
bbb