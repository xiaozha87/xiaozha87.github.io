---
title: windows 的cmd设置代理的问题
date: 2018-08-27 10:18:30
tags:
- Github
category: Github
---

# 电脑通过设置代理来访问外网，则需要为cmd设置代理：

## 1、首先打开 cmd （win + R，输入 cmd，然后按 enter 键）

## 2、输入以下命令
```bash
set http_proxy=http://127.0.0.1:1189
set https_proxy=http://127.0.0.1:1189
```
其中":"后面的为自己的代理服务器的地址，可自行查阅
注意，代理服务器的地址的查看：谷歌浏览器——设置——高级——打开代理设置——局域网设置——复制代理服务器地址即可

## 3、如果你的代理服务器要求用户名和密码的话，那么还需要：
```bash
set http_proxy_user=
set http_proxy_pass=
```
## 4、设置完成之后就可以在 cmd 下正常使用网络了。
