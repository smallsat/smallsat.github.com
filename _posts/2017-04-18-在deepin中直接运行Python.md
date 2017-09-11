---
layout: post
title: '直接在deepin中运行.py'
date: 2017-04-18
author: smallman
tags: python deepin
---


在 `Deepin` 中直接运行 `.py` 的文件与其他系统中略有不同, 在其他系统中 *可能* 在第一行添加：
```cmd
#!/usr/bin/env python3
```
但是在 `Deepin` 中直接添加是无效的，需要更改下
```cmd
#! /usr/bin/python3
```
添加完毕就可以双击运行 `.py` 的文件了