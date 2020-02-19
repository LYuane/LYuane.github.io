---
layout:     post
title:      google hacker
subtitle:
date:       2018-06-18
author:     Yuane
header-img: img/post.jpg
catalog: true
tags:
    - CTF
    - web安全
---



## 0x01高级操作符

- 高级搜索 **inurl：xxx**

可以查询url中有xxx的网页，可以用来找后台登录页面如：

```
inurl:login_admin.asp
```

可以用来找上传页面如：

```
inurl:upload_soft.asp
```

- 高级搜索 **intext：xxx**

把网页中的正文内容中的某个字符做为搜索条件,可以用来查看网站目录如：

```
intext：to paremt directory
```

- 高级搜索 **Filetype：xxx**

搜索指定文件，可以用来查数据库如：

```
Filetype：mdb
```

- 高级搜索 **site：xxx**
- intitle:搜索网页标题中是否有所输入字符
- cache:
  搜索google里关于某些内容的缓存
- define:
  搜索某个词语的定义,搜索:define:hacker,将返回关于hacker的定义.
- info:
  查找指定站点的一些基本信息.
- link:
  例如搜索:inurl:www.4ngel.net可以返回所有和www.4ngel.net做了链接的URL.
- site:

这个也很有用,例如:site:www.4ngel.net.将返回所有和4ngel.net这个站有关的URL.

related：搜索所有链接到指定链接的相关URL



+把google可能忽略的字列如查询范围
-把某个字忽略
~同意词
.单一的通配符
*通配符，可代表多个字母
""精确查询



## 0x02 组合使用法（示例）：

1. 搜索相关页面（或数据库文件）
   Site:xxx Filetype:mdb
   Site:xxx intext: to parent directory
   Site:xxx intext: to parent directory Filetype:mdb
2. 下载到数据库后，登陆后台管理
   Site:xxx intitle:管理
   Site:xxx inurl:login.asp
3. 直接找上传点
   Site:xxx inurl:upload.asp

*注意：robot.txt是针对搜索引擎机器人的存文本文件，可以说明不对robot访问的部分*

## 0x03 相关链接

[Google Hack技巧——不用暴力也可以取得密码]: https://blog.csdn.net/Jeanphorn/article/details/44907737	"Google Hack技巧——不用暴力也可以取得密码"
[Google Hacking————你真的会用Google吗？]: https://zhuanlan.zhihu.com/p/22161675	"Google Hacking————你真的会用Google吗？"
[Google Hacking：教你找女神]: https://www.freebuf.com/articles/web/32610.html

[Google高级搜索hacker技巧](http://blog.whiterabbitxyj.com/2017/10/16/googlesearchskill/)

