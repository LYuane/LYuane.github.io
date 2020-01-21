---
layout:     post
title:      google hacker
subtitle:
date:       2018-10-07
author:     Yuane
header-img: img/post.jpg
catalog: true
tags:
    - CTF
    - web安全
---



### 0x01高级操作符

intext:把网页中的正文内容中的某个字符做为搜索条件



intitle:搜索网页标题中是否有所输入字符

cache:
搜索google里关于某些内容的缓存

define:
搜索某个词语的定义,搜索:define:hacker,将返回关于hacker的定义.

filetype:
搜索指定类型的文件.例如输入:filetype:asp.将返回所有以asp结尾的文件的URL，可以与其他操作符混合使用

info:
查找指定站点的一些基本信息.

inurl:
搜索输入字符是否存在于URL中.可以联合site指定来找后台、fck之类，可以与其他操作符混合使用，可单独使用

link:
例如搜索:inurl:www.4ngel.net可以返回所有和www.4ngel.net做了链接的URL.



site:

这个也很有用,例如:site:www.4ngel.net.将返回所有和4ngel.net这个站有关的URL.

related：搜索所有链接到指定链接的相关URL





+把google可能忽略的字列如查询范围
-把某个字忽略
~同意词
.单一的通配符
*通配符，可代表多个字母
""精确查询



### 0x02相关链接

[Google Hack技巧——不用暴力也可以取得密码]: https://blog.csdn.net/Jeanphorn/article/details/44907737	"Google Hack技巧——不用暴力也可以取得密码"
[Google Hacking————你真的会用Google吗？]: https://zhuanlan.zhihu.com/p/22161675	"Google Hacking————你真的会用Google吗？"
[Google Hacking：教你找女神]: https://www.freebuf.com/articles/web/32610.html

