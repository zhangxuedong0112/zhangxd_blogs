---
title: MongoDB 搭建
categories: "MongoDB" #文章分類目錄 可以省略
date: 2017-07-18 17:55:36
tags: "DB"
---

## mongodb的搭建步骤
<!--![Alt text](../../zhangxd/img/zhangxuedong.jpg)-->
![Alt text](/img/mongo.jpg)

<!--more-->

``` bash
1. wget https://fastdl.mongodb.org/linux/mongodb-linux-i686-2.6.7.tgz?_ga=1.68265944.858401362.1421216907
2. tar -zxvf mongodb-linux-i686-2.6.7.tgz\?_ga\=1.68265944.858401362.1421216907 
3. mv mongodb-linux-i686-2.6.7 mongodb
4. rm mongodb-linux-i686-2.6.7.tgz\?_ga\=1.68265944.858401362.1421216907 
5. mkdir -p /data/mongo/logs
6. mkdir -p /data/mongo/db
7. cd mongodb/
8. cd bin/
9. vi mongodb.conf  
10. dbpath = /data/mongo/db #数据文件存放目录
logpath = /data/mongo/logs/mongodb.log #日志文件存放目录
port = 27017  #端口
fork = true  #以守护程序的方式启用，即在后台运行
nohttpinterface = true
auth=true
11. cp mongodb.conf /etc/mongodb.conf
12. rm mongodb.conf 
13. sudo ln -s /home/zhangxd/work/mongodb/mongodb/bin/mongod /usr/bin/mongod
14. ln -s /home/zhangxd/work/mongodb/mongodb/bin/mongo /usr/bin/mongo
15. sudo yum install glibc.i686
16. yum whatprovides libstdc++.so.6   http://www.linuxidc.com/Linux/2013-04/82494.htm
17. sudo yum install libstdc++-4.4.7-16.el6.i686  --setopt=protected_multilib=false//看16步缺少的东西就安装那个
18. mongod -f /etc/mongodb.conf 
19. vi /etc/rc.d/rc.local 添加自启动  mongod -f /etc/mongodb.conf 
20. db.addUser('tank1','test'); //添加用户,给哪个db添加就去哪个db下
     db.auth(“tank1”,”test”)//验证，使用哪个db去哪个db


use admin
db.addUser('admin','abcd-1234')
db.auth('admin','abcd-1234')

use hplx_mongo
db.test.insert({"name":"test"})
db.addUser('hplx_admin','abcd-1234')
db.auth('hplx_admin ','abcd-1234')


3.4.2添加用户
use test
db.createUser({user:"zhangxd",pwd:"zhangxd",roles:[{role:"readWrite",db:"test"}]})
```