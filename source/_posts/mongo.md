---
title: mongodb安装部署
tags:
  - mongodb
  - 数据库
categories:
  - 数据库
---




### 1.部署

#### a. 单机部署

**1.配置MongoDB的yum源**

创建yum源文件：
```
vim /etc/yum.repos.d/mongodb-org-3.4.repo
```
添加以下内容：
```
[mongodb-org-3.4]  
name=MongoDB Repository  
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/  
gpgcheck=1  
enabled=1  
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```
这里可以修改 gpgcheck=0, 省去gpg验证

安装之前先更新所有包 （可选操作）：
```
yum update 
```
**2.安装MongoDB**
安装命令：
```
yum -y install mongodb-org
```
安装完成后

查看mongo安装位置 
```
whereis mongod
```
查看修改配置文件 ：
```
vim /etc/mongod.conf
```
**3.启动MongoDB** 
启动mongodb ：
```
systemctl start mongod.service
```
停止mongodb ：
```
systemctl stop mongod.service
```

查到mongodb的状态：
```
systemctl status mongod.service
```
  **4.外网访问需要关闭防火墙：**
CentOS 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙。
关闭firewall：
```
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
**5.设置开机启动**
```
systemctl enable mongod.service  
```
**6.启动Mongo shell**
命令：mongo 

**7.设置mongodb远程访问：**
编辑mongod.conf注释bindIp,并重启mongodb.
```
vim /etc/mongod.conf
```
重启mongodb：
```
systemctl restart mongod.service
```
#### b. 分布式集群部署

#### 1.mongodb安装包下载

  到https://www.mongodb.org/dl/linux/x86_64下载

  mongodb-linux-x86_64-v3.4-latest.gz

#### 2.建数据存储目录

/.../data/mongodb/mongos/{db,log}

/.../data/mongodb/configServer/{db,log}

/.../data/mongodb/shard1/{db,log}

/.../data/mongodb/shard2/{db,log}

#### 3.写配置文件**mongo_node.conf**

/.../data/mongodb/conf/**mongo_node.conf**

```
storage:
    engine: wiredTiger
    directoryPerDB: true
    journal:
        enabled: true
systemLog:
    destination: file
    logAppend: true
operationProfiling:
  slowOpThresholdMs: 10000
replication:
    oplogSizeMB: 10240
processManagement:
    fork: true
net:
    bindIp: 0.0.0.0
    http:
      enabled: true
security:
    authorization: "enabled"
```

#### 4.写配置文件mongos.conf

/.../data/mongodb/conf/mongos.conf

```
systemLog:
    destination: file
    logAppend: true
processManagement:
    fork: true
net:
    http:
      enabled: false
```

#### 5.创建keyfile文件

/.../data/mongodb/keyfile/mongo.key

```
mkdir keyfile
openssl rand -base64 756 > mongo.key
chmod 400 mongo.key
mv mongo.key keyfile
```

#### 6.远程发送keyfile文件到其他节点

需要root权限

```
sudo scp mongo.key @test2:/home/test/apps/data/mongodb/keyfile
```

#### 7.集群搭建

##### 1）编写配置服务启动脚本

a.配置服务脚本config_start.sh

```
WORK_DIR=/home/test/apps/data/mongodb
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongo_node.conf
MONGOD=/home/test/apps/mongodb3.4.23/bin/mongod
DBPATH=$WORK_DIR/configServer/db
LOGPATH=$WORK_DIR/configServer/log
$MONGOD --port 27017 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $DBPATH --pidfilepath $WORK_DIR/configServer/db.pid --logpath $LOGPATH/mongod.log --config $CONFFILE
```

b.副本集初始化

随便进入一个mongo客户端

./mongo 

```
   var conf = {
  _id : "configReplSet",
  members : [
    { _id : 1, host : "192.168.2.132:27017" },
    { _id : 2, host : "192.168.2.133:27017" },
    { _id : 3, host : "192.168.2.134:27017" }
  ]
}

rs.initiate(conf)
```

##### 2) 编写分片脚本shard1_start.sh和shard2_start.sh

a.编写shard1脚本shard1_start.sh

```
WORK_DIR=/home/test/apps/data/mongodb
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongo_node.conf
MONGOD=/home/test/apps/mongodb3.4.23/bin/mongod

echo "start shard1 replicaset"

$MONGOD --port 27001 --shardsvr --replSet shard1 --keyFile $KEYFILE --dbpath $WORK_DIR/shard1/db --pidfilepath $WORK_DIR/configServer/db.pid --logpath $WORK_DIR/shard1/log/mongod.log --config $CONFFILE
```

b.编写shard2脚本shard2_start.sh

```
WORK_DIR=/home/test/apps/data/mongodb
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongo_node.conf
MONGOD=/home/test/apps/mongodb3.4.23/bin/mongod

echo "start shard1 replicaset"

$MONGOD --port 27002 --shardsvr --replSet shard2 --keyFile $KEYFILE --dbpath $WORK_DIR/shard2/db --pidfilepath $WORK_DIR/configServer/db.pid --logpath $WORK_DIR/shard2/log/mongod.log --config $CONFFILE
```

c.分别在三台机器执行两个脚本

d.初始化副本集

进入一个客户端

./mongo --port 27001 --host 127.0.0.1

```
cfg={
    _id:"shard1", 
    members:[
        {_id:0, host:'192.168.2.132:27001'},
        {_id:1, host:'192.168.2.133:27001'}, 
        {_id:2, host:'192.168.2.134:27001'}
    ]};
rs.initiate(cfg);
```

进入另一个

./mongo --port 27002 --host 127.0.0.1

```
cfg={
    _id:"shard2", 
    members:[
        {_id:0, host:'192.168.2.132:27002'},
        {_id:1, host:'192.168.2.133:27002'}, 
        {_id:2, host:'192.168.2.134:27002'}
    ]};
rs.initiate(cfg);
```



##### 3.编写路由脚本

a.编写mongos_start.sh

```
WORK_DIR=/home/test/apps/data/mongodb
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongos.conf
MONGOS=/home/test/apps/mongodb3.4.23/bin/mongos

echo "start mongos instances"
$MONGOS --port 25001 --configdb configReplSet/192.168.2.132:27017,192.168.2.133:27017,192.168.2.134:27017 --keyFile $KEYFILE --pidfilepath $WORK_DIR/configServer/db.pid --logpath $WORK_DIR/mongos/log/db.log --config $CONFFILE

```

b.进入客户端添加分片

./mongo --port 25001 --host 127.0.0.1

```
sh.addShard("shard1/192.168.2.132:27001")

sh.addShard("shard2/192.168.2.134:27002")

```

##### 5.初始化用户

接入其中一个mongos实例，添加管理员用户

```
use admin
db.createUser({
    user:'admin',pwd:'admin',
    roles:[
        {role:'clusterAdmin',db:'admin'},
        {role:'userAdminAnyDatabase',db:'admin'},
        {role:'dbAdminAnyDatabase',db:'admin'},
        {role:'readWriteAnyDatabase',db:'admin'}
]})
```

当前admin用户具有集群管理权限、所有数据库的操作权限。
需要注意的是，在第一次创建用户之后，localexception不再有效，接下来的所有操作要求先通过鉴权。

```
use admin
db.auth('admin','admin')
```

检查集群状态

```
mongos> sh.status()
--- Sharding Status --- 
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("5da5796d1af1c6d6138fad3f")
  }
  shards:
        {  "_id" : "shard1",  "host" : "shard1/192.168.2.132:27001,192.168.2.133:27001,192.168.2.134:27001",  "state" : 1 }
        {  "_id" : "shard2",  "host" : "shard2/192.168.2.132:27002,192.168.2.133:27002,192.168.2.134:27002",  "state" : 1 }
  active mongoses:
        "3.4.23-8-g5e9b683bee" : 3
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
NaN
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
```

#### 8 .数据操作

在案例中，创建testpuser用户、为数据库实例testb启动分片。

```
test
db.createUser({user:'testuser',pwd:'123456',roles:[{role:'dbOwner',db:'test")
```

创建集合book，为其执行分片初始化。

```
use appdb
db.createCollection("book")
db.device.ensureIndex({createTime:1})
sh.shardCollection("appdb.book", {bookId:"hashed"}, false, { numInitialChunks: 4} )
```

继续往device集合写入1W条记录，观察chunks的分布情况

```
use test
var cnt = 0;
for(var i=0; i<100; i++){
    var dl = [];
    for(var j=0; j<100; j++){
        dl.push({
                "bookId" : "BBK-" + i + "-" + j,
                "type" : "Revision",
                "version" : "IricSoneVB0001",
                "title" : "Jackson's Life",
                "subCount" : 10,
                "location" : "China CN Shenzhen Futian District",
                "author" : {
                      "name" : 50,
                      "email" : "RichardFoo@yahoo.com",
                      "gender" : "female"
                },
                "createTime" : new Date()
            });
      }
      cnt += dl.length;
      db.book.insertMany(dl);
      print("insert ", cnt);
}
```

执行**db.book.getShardDistribution()**,输出如下：

```
mongos> db.book.getShardDistribution();

Shard shard1 at shard1/192.168.2.132:27001,192.168.2.133:27001,192.168.2.134:27001
 data : 1.32MiB docs : 4961 chunks : 2
 estimated data per chunk : 680KiB
 estimated docs per chunk : 2480

Shard shard2 at shard2/192.168.2.132:27002,192.168.2.133:27002,192.168.2.134:27002
 data : 1.34MiB docs : 5039 chunks : 2
 estimated data per chunk : 690KiB
 estimated docs per chunk : 2519

Totals
 data : 2.67MiB docs : 10000 chunks : 4
 Shard shard1 contains 49.61% data, 49.61% docs in cluster, avg obj size on shard : 280B
 Shard shard2 contains 50.38% data, 50.39% docs in cluster, avg obj size on shard : 280B
```

#### 各方案的优劣

##### 单机模式：

优点：

a.部署容易，数据不需要复制备份，节省资源，适合少量数据的存取

b.安全稳定，受网络影响小，安全性好

c.排障容易，只有一台机器也容易找到故障来源

缺点：

a.可靠性差，机器故障容易丢失数据

b.扩展性差，升级配置需要的成本高

c.性能差，高并发时容易宕机



##### 集群模式：

优点：

a.可靠，一台服务器宕机，并不影响系统运行

b.可扩展，随时可以增加机器；

c.资源共享，共享数据是必不可少的应用，如银行，预订系统。

d.更快的速度：多台计算机的运算速度远大于一台。

e.高性价比：相比纵向扩展硬件配置，节省了开支

缺点：

a.故障排除，产生故障时需要花费的时间更多

b.网络影响，受网络限制更大，集群本身机器之间的数据传输依赖网络传输

c.安全性，资源共享会导致安全风险

### 2.性能测试

服务器配置：

内存：64g

磁盘分区：11t

系统信息：

LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: CentOS
Description:    CentOS Linux release 7.2.1511 (Core) 
Release:        7.2.1511
Codename:       Core



#### a. 单机

##### （1 读/写速度

a.文件读写

```
导入数据
2019-10-16T18:39:45.266+0800    imported 100000 documents
导出数据
2019-10-16T18:39:59.048+0800    connected to: 127.0.0.1:25001
2019-10-16T18:40:00.036+0800    [#.......................]  test.book  8000/100000  (8.0%)
2019-10-16T18:40:01.036+0800    [#######.................]  test.book  32000/100000  (32.0%)
2019-10-16T18:40:02.036+0800    [###########.............]  test.book  48000/100000  (48.0%)
2019-10-16T18:40:03.036+0800    [#################.......]  test.book  72000/100000  (72.0%)
2019-10-16T18:40:04.036+0800    [#####################...]  test.book  88000/100000  (88.0%)
2019-10-16T18:40:04.268+0800    [########################]  test.book  100000/100000  (100.0%)
2019-10-16T18:40:04.268+0800    exported 100000 records
100000条数据耗时： 6 s
1000w条数据：134 s
写一亿条： 4514 s
读一亿条：内存溢出
```

b.api请求读写

单条数据请求写10000次耗时37s

单条数据请求读10000次耗时40s

##### （2 容量大小

```
    "db" : "test",
    "collections" : 1,
    "views" : 0,
    "objects" : 337125,
    "avgObjSize" : 234.4636559139785,
    "dataSize" : 79043560,
    "storageSize" : 11988992,
    "numExtents" : 0,
    "indexes" : 1,
    "indexSize" : 3481600,
    "ok" : 1
```

}

##### （3 资源使用

内存：268m

#### b. 集群

##### （1 读/写速度

a.文件读写

```
导入数据
2019-10-16T18:16:55.532+0800    connected to: 127.0.0.1:25001
2019-10-16T18:16:57.651+0800    imported 100000 documents
100000条数据耗时： 2 s
导出数据
2019-10-16T18:17:04.586+0800    connected to: 127.0.0.1:25001
2019-10-16T18:17:05.564+0800    [#.......................]  test.book  8000/100000  (8.0%)
2019-10-16T18:17:06.564+0800    [#######.................]  test.book  32000/100000  (32.0%)
2019-10-16T18:17:07.564+0800    [###########.............]  test.book  48000/100000  (48.0%)
2019-10-16T18:17:08.564+0800    [#################.......]  test.book  72000/100000  (72.0%)
2019-10-16T18:17:09.564+0800    [#######################.]  test.book  96000/100000  (96.0%)
2019-10-16T18:17:09.760+0800    [########################]  test.book  100000/100000  (100.0%)
2019-10-16T18:17:09.760+0800    exported 100000 records
100000条数据耗时： 5 s
1000w条耗时322 s
10w条耗时336s
```

b.api请求读写

单条数据请求写10000次耗时39s

单条数据请求写100000次耗时386s

单条数据请求读10000次耗时51s

##### （2 容量大小

导入数据2500w条的时候，导入速度变慢，但是api请求的速度不变

##### （3 资源使用

​	内存：1.8g

### 3.常用的数据存储方式

mongodb存储形式为文档形式，支持xml、json、bson

mongodb的数据格式为任意类型的键值对，并且可以键值对嵌套在value中

数据存储方式为数据文件如：

collection-3-4415151890359181580.wt

以及索引文件：

index-4-4415151890359181580.wt



### 4.数据安全

#### a. 备份

数据备份：

mongodump -h host:port  -u user -p pwd -d test -o path

如：./mongodump -h 127.0.0.1:25001 -u testuser -p 123456 -d test -o /data02/fw/bac

数据恢复：

mongorestore -h host:port  -u user -p pwd -d test -directoryperdb path



#### b. 主从

mongodb采用主从复制的方式保证数据安全，我配置了三个节点，自动选举出一个主节点，两个从节点，共有三个备份

主节点：192.168.2.203：26001

从节点：192.168.2.203：26002，192.168.2.203：26003

### 5.权限控制

#### a. 数据分区控制

mongodb没有分区，但是分布式mongodb可以分片

我配置了两个分片shard1和shard2

shard1：192.168.2.203:28001,92.168.2.203:28002,92.168.2.203:28003

shard2:    192.168.2.203:28004,92.168.2.203:28005,92.168.2.203:28006

分片可以减少单台服务器存储压力，mongodb会将数据在每个分片间自动均衡

#### b. 用户授权使用

刚刚安装的mongodb是没有权限限制的，在创建了管理员账户后，所有的操作都需要用户验证，所以需要再创建一个可读可写的普通用户

使用分布式mongodb时，都通过mongos实例对数据进行操作

./mong --port 25001

创建管理员

```
use admin
db.createUser({
    user:'admin',pwd:'admin',
    roles:[
        {role:'clusterAdmin',db:'admin'},
        {role:'userAdminAnyDatabase',db:'admin'},
        {role:'dbAdminAnyDatabase',db:'admin'},
        {role:'readWriteAnyDatabase',db:'admin'}
]})

```

身份验证

```
use admin
db.auth('admin','admin')

```



创建普通用户

```
db.createUser({user:'testuser',pwd:'123456',roles:[{role:'dbOwner',db:'test")

```







#### c. 网络接入控制

**一，访问控制的参数**

**1，绑定IP地址**

```
配置文件：
net:
    bindIp: 0.0.0.0
mongod 参数：**--bind_ip**`` <ip address>
访问ip：192.168.2.203

```

默认值是所有的IP地址都能访问，该参数指定MongoDB对外提供服务的绑定IP地址，用于监听客户端 Application的连接，客户端只能使用绑定的IP地址才能访问mongod，其他IP地址是无法访问的。

**2，设置监听端口**

```
mongod 参数：**--port** <port>
25001/25001/25003

```

MongoDB 默认监听的端口是27017，该参数显式指定MongoDB实例监听的TCP 端口，只有当客户端Application连接的端口和MongoDB实例监听的端口一致时，才能连接到MongoDB实例。

**3，启用用户验证**

```
mongod 参数：**--auth** 
配置文件：
security:
    authorization: "enabled"
admin：admin
testuser:123456

```

**4，权限认证**

```
mongo 参数：**--username** <username>, -u <username>
mongo 参数：**--password** <password>, -p <password>
mongo 参数：**--authenticationDatabase** <dbname>  指定创建User的数据库；在特定的数据库中创建User，该DB就是User的authentication database。

```

