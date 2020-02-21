---
layout:     post
title:      提高Github Clone速度
date:       2018-03-19
author:     Yuane
header-img: img/post.jpg
catalog: true
tags:
    - 效率

---



## Method 1 ：SS+系统内置代理

用 git 内置代理，直接走系统中运行的代理工具中转，比如，你的 SS 本地端口是 1080（一般port均为1080），那么可以如下方式走代理：

P.S. 仅启动shadowsocks不起作用，需同时配置git内置代理

```
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

编辑.gitconfig文件

> .gitconfig文件地址（以windows为例）
> C:\Users\username\.gitconfig



## Method 2 ：修改系统hosts

git clone或者git push特别慢，可能是因为 http://github.com
http://github.global.ssl.fastly.Net
这两个域名被限制了。
那么可以在hosts文件里进行绑定映射。

1.输入相应域名，使用[DNS检测](http://tool.chinaz.com/)
2.找TTL值最小的响应ip
3.修改本地hosts文件
示例：在末尾添加

```
151.101.72.249 http://global-ssl.fastly.Net
192.30.253.112 http://github.com
```