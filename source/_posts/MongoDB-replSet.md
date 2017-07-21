---
title: MongoDB-ReplSet集群
categories: "MongoDB" #文章分類目錄 可以省略
date: 2017-07-21 10:17:27
tags: "DB"
---

<!-- ## Mongodb-ReplSet集群的搭建步骤 -->
![Alt text](/img/mongo.jpg)

<!--more-->

``` bash
1. port = 27017        //监听端口  
2. fork = true         //后台运行  
3. pidfilepath = /var/run/mongodb/mongodb.pid    //进程PID文件  
4. logpath = /var/log/mongodb/mongodb.log        //日志文件  
5. dbpath =/var/lib/mongodb           //db存放目录  
6. journal = true                   //存储模式  
7. nohttpinterface = true           //禁用http  
8. directoryperdb=true              //一个数据库一个文件夹  
9. logappend=true                  //追加方式写日志  
10. replSet=repmore                 //集群名称，自定义  
11. oplogSize=1000                  //oplog大小  

sudo vi /etc/mongodb.27017.conf
dbpath = /data/mongo/27017/db #数据文件存放目录
logpath = /data/mongo/logs/mongodb.log #日志文件存放目录
port = 27017  #端口
fork = true  #以守护程序的方式启用，即在后台运行
nohttpinterface = true
replSet=repmore
directoryperdb=true
logappend=true
auth=false  //不关闭没法同步

sudo vi /etc/mongodb.27018.conf
dbpath = /data/mongo/27018/db #数据文件存放目录
logpath = /data/mongo/logs/mongodb.log #日志文件存放目录
port = 27018  #端口
fork = true  #以守护程序的方式启用，即在后台运行
nohttpinterface = true
replSet=repmore
directoryperdb=true
logappend=true
auth=false

sudo vi /etc/mongodb.27019.conf
dbpath = /data/mongo/27019/db #数据文件存放目录
logpath = /data/mongo/logs/mongodb.log #日志文件存放目录
port = 27019  #端口
fork = true  #以守护程序的方式启用，即在后台运行
nohttpinterface = true
replSet=repmore
directoryperdb=true
logappend=true
auth=false

同一台机器启动 需要使用numactl命令
numactl --interleave=all mongod -f /etc/mongodb.27017.conf
numactl --interleave=all mongod -f /etc/mongodb.27018.conf
numactl --interleave=all mongod -f /etc/mongodb.27019.conf

连接其中一台
mongo 127.0.0.1:27017
use admin
config={_id:"repmore",members:[{_id:0,host:"127.0.0.1:27017"},{_id:1,host:"127.0.0.1:27018"},{_id:2,host:"127.0.0.1:27019"}]};
rs.initiate(config); 初始化
在这里要注意，rs.initiate初始化也是要一定时间的，刚执行完rs.initiate，我就查看状态，从服务器的stateStr不是SECONDARY，而是stateStr : "STARTUP2"，等一会就好了。
rs.status() 产看状态
rs.remove("127.0.0.1:27019")//删除
rs.add("127.0.0.1:27019")//添加 

测试
主
1. repmore:PRIMARY> show dbs;  
2. local    1.078125GB  
3. repmore:PRIMARY> use test  
4. switched to db test  
5. repmore:PRIMARY> db.test.insert({'name':'tank','phone':'12345678'});  
6. repmore:PRIMARY> db.test.find();  
7. { "_id" : ObjectId("52af64549d2f9e75bc57cda7"), "name" : "tank", "phone" : "12345678" }
从
1. [root@localhost mongodb]# mongo 127.0.0.1:27018   //连接  
2. MongoDB shell version: 2.4.6  
3. connecting to: 127.0.0.1:27018/test  
4. repmore:SECONDARY> show dbs;  
5. local    1.078125GB  
6. test    0.203125GB  
7. repmore:SECONDARY> db.test.find();     //无权限查看  
8. error: { "$err" : "not master and slaveOk=false", "code" : 13435 }  
9. repmore:SECONDARY> rs.slaveOk();       //从库开启  
10. repmore:SECONDARY> db.test.find();     //从库可看到主库刚插入的数据  
11. { "_id" : ObjectId("52af64549d2f9e75bc57cda7"), "name" : "tank", "phone" : "12345678" }  
12. repmore:SECONDARY> db.test.insert({'name':'zhangying','phone':'12345678'});   //从库只读，无插入权限  
13. not master  

db.createUser({user:'sleliao',pwd:'sleliao',roles:['userAdminAnyDatabase','dbAdminAnyDatabase']}) //可登陆所有db的用户
db.createUser({user:"admin",pwd:"admin",roles:[{role:"root",db:"admin"}]}) //超级管理员

 配置完成之后启动认证
#auth=false
keyFile = /home/zhangxd/work/mongodb/keyfiletest
生成key
 openssl rand -base64 90 > openssl rand -base64 90 > /root/software/mongodb/keyfiletest
 scp  /root/software/mongodb/keyfile root@192.168.91.133:/root/software/mongodb/ 复制到其他节点
 sudo chmod 600 ./keyfiletest  给600权限
然后重启3台服务器，去客户端测试一下 ok了


安装mongo https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/
mongodb replica set 多服务器 高可用 配置 详解 http://blog.51yip.com/nosql/1580.html
副本集认证http://www.cnblogs.com/xiaoit/p/4522218.html
sudo chmod 600 ./keyfiletest

查看读写日志
切到想看日志的db
use test
db.setProfilingLevel(2)  设置读写log
profile级别有三种：
0：不开启
1：记录慢命令，默认为大于100ms
2：记录所有命令
3、查询profiling记录

db.getCollection('system.profile').find({}).sort({ts:-1})


ERROR: child process failed, exited with error number 45
删掉.lock sudo rm /var/lib/mongo/mongod.lock  sudo rm /data/mongo/db/mongod.lock  
修复db  sudo mongod --dbpath /data/mongo/db --repair 
然后重启试试 sudo mongod -f /etc/mongodb.conf
mongod -f /etc/mongodb.conf --smallfiles

```