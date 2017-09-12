---
layout: post
title: '在Deepin中配置PAC'
date: 2017-04-18
categories: Linux
author: smallsat
tags: deepin PAC Linux
---


> 在编码过程中常常需要使用 `Google` 查找资料，之前一直使用免费的SS，但是经常更新密码，一咬牙一跺脚，还是自己买一个。买的是便宜货，每个月有流量限制，发现在 `Deepin` 中的 `ss-qt5` 不能使用PAC，为了节省流量，安装了PAC。


### 1. 安装pip
运行

```bash
sudo apt-get install python-pip
```

### 2. 安装genpac

```bash
pip install genpac  #安装
pip install --upgrade genpac  #更新
pip uninstall genpac #卸载
```

### 3.下载
为了方便管理，创建了一个文件夹

```bash
mkdir ~/shadowsocks
cd shadowsocks
touch autoproxy.pac
```

下载PAC

``` bash
~/.local/bin/genpac -p "SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" --output="autoproxy.pac" --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt" --user-rule-from="autoproxy.pac"
```

### 4.设置代理

点击 设置→网络→代理→自动，在URL处填写`file:///home/{user}/shadowsocks/autoproxy.pac` 保存。就可以了。
