---
layout:     post
title:      利用docker快速搭建FTP服务器
date:       2018-10-23
author:     Yuane
header-img: img/home-bg.jpg
catalog: true
tags:
    - docker

---

### 0x00 

docker压缩包已上传至github

### 0x01 导入镜像

```
cat ftp.rar | docker import - ftp:tyz      //[REPOSITORY]:[tag]
```



### 0x02 启动容器

```
docker run -dt --name ftp -e "PUBLICHOST=localhost" --privileged=true -v /home/ftpusers/robin:/home/ftpusers/www ftp:tyz bash

说明：

　　使用绑定IP为192.168.0.3，如果是公开FTP的话，可以不写IP。

　　FTP用户计算方式为“(最大端口号-最小端口号) / 2”。我这里修改为可以满足100个用户同时连接登陆

　　做了个目录映射，把本机的/home/ftpusers/robin目录映射到pure-ftp的/home/ftpusers/www下
```

### 0x03 运行FTP

```
1.进入docker容器
docker exec -it ftp bin/bash

2.在容器内执行命令
/usr/sbin/pure-ftpd -c 100 -C 100 -l puredb:/etc/pure-ftpd/pureftpd.pdb -E -j -R -P $PUBLICHOST -p 30000:30209 &
```

### 0x04 测试

```
ftp -p [IP] [port]
如果没变的话应该是   ftp -p 192.168.0.3 21

预设两个用户名/密码     www/www     test/test
```

### 0x05 从dockerhub拉取

```
docker pull stilliard/pure-ftpd:hardened

docker run -d --name ftp1 -p 2121:21 -p 30000-30009:30000-30009 -e FTP_USER_NAME=test -e FTP_USER_PASS=test -e FTP_USER_HOME=/home/test stilliard/pure-ftpd:hardened
```



测试

```
ftp -p ip 21

用户/密码   test/test

上传文件   put [本地路径]  [上传路径]
```



实现匿名访问

```
docker exec -it [content-id] bash

$ cd /etc/pure-ftpd/conf
$ vi NoAnonymous
改为no

$ service vsftpd restart
```



如果出现

```
bash: vi: command not found
```

解决办法

```
apt-get update
apt-get install vim
vim NoAnonymous
```

