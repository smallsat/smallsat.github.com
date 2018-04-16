---
layout: post
title: '配置workerman的wss'
date: 2017-10-26
author: smallsat
categories: wss workerman
tags: wss workerman websocket
---

在使用`workerman`的过程中，需要配置`wss`，在这期间踩了很多的坑，这里详细记录下过程
# 配置`workerman`的监听
    这里踩过的坑比较少，还比较好理解。代码是基于 `thinkphp`
    
1. $this->stock = "127.0.0.1:2346";
这里的端口可以更改
2. 启动服务

# 配置`Nginx`的代理配置
如果只是使用`ws`这一步已经可以了。但是要使用`wss`就不行了。
`wss`是基于`websocket`+`ssl`
1. 配置`ssl`信息
```ini
    ssl on;   #开启ssl
    ssl_certificate    /www/server/nginx/conf/cert/rtes.bingplus.com.pem; 
    ssl_certificate_key    /www/server/nginx/conf/cert/rtes.bingplus.com.key;
    
    #代理设置
  location /wss     #这里是将所有的https://域名/wss访问都转发到端口
      {
        proxy_pass http://127.0.0.1:2346;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header X-Real-IP $remote_addr;
      }
```