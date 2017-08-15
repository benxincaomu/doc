# mongo集群搭建

## 准备

创建dbpath

```shell
mkdir -p /usr/soft/mongo/mongodb/db/config
mkdir -p /usr/soft/mongo/mongodb/db/27011
mkdir -p /usr/soft/mongo/mongodb/db/27012
mkdir -p /usr/soft/mongo/mongodb/db/27013

```

创建logpath

```shell
mkdir -p /usr/soft/mongo/mongodb/logs/config
mkdir -p /usr/soft/mongo/mongodb/logs/27011
mkdir -p /usr/soft/mongo/mongodb/logs/27012
mkdir -p /usr/soft/mongo/mongodb/logs/27013
mkdir -p /usr/soft/mongo/mongodb/logs/route
```

## config server

### 命令行启动

```shell
./bin/mongod --configsvr --replSet test --dbpath /usr/soft/mongo/mongodb/db/config --port 27010 --logpath /usr/soft/mongo/mongodb/logs/config/log --fork
```

可启动多个配置服务器

### 初始化

 使用mongo客户端连接任意一台配置服务器

```shell
./bin/mongo --port 27010
```

执行以下命令，多台配置服务器时在members中添加响应的配置

```javascript
rs.initiate({_id:"test",configsvr:true,members:[{_id:0,host:"127.0.0.1:27010"}]})
```



## shard server

### 命令行启动

```shell
./bin/mongod --shardsvr --replSet testSet --port 27011 --dbpath /usr/soft/mongo/mongodb/db/27011 --logpath /usr/soft/mongo/mongodb/logs/27011/log --fork --nojournal  

./bin/mongod --shardsvr --replSet testSet --port 27012 --dbpath /usr/soft/mongo/mongodb/db/27012 --logpath /usr/soft/mongo/mongodb/logs/27012/log --fork --nojournal  

./bin/mongod --shardsvr --replSet testSet0 --port 27013 --dbpath /usr/soft/mongo/mongodb/db/27013 --logpath /usr/soft/mongo/mongodb/logs/27013/log --fork --nojournal 
```

### 初始化

登陆27011,执行以下命令，即可对27011与27012进行初始化

```shell
./bin/mongo --port 27011
#下面在mongo命令行中执行
config={_id:"testSet", members:[{_id:0,host:"127.0.0.1:27011"},{_id:1,host:"127.0.0.1:27012"}]}
rs.initiate(config);
```

此时，27011和27012则作为一个分片存在,而且这两个服务器中中的数据将保持一致。

登陆27013，执行命令即可将27013作为另外一个分片服务器

```shell
./bin/mongo --port 27013
#下面在mongo命令行中执行
config={_id:"testSet0", members:[{_id:0,host:"127.0.0.1:27013"}]}
rs.initiate(config);
```



## route server

### 命令行启动

```shell
./bin/mongos --configdb test/127.0.0.1:27010 --port 1999 --logpath /usr/soft/mongo/mongodb/logs/route/log --fork
```



### 添加分片到路由

登陆路由服务器，

```shell
./bin/mongo --port 1999
#下面在mongo命令行中执行
sh.addShard("testSet/127.0.0.1:27011,127.0.0.1:27012")
sh.addShard("testSet0/127.0.0.1:27013")
sh.enableSharding("test")
sh.shardCollection("test.testCollection", { id: "hashed"})
```

至此，一个mongodb的分片集群服务搭建完成。访问路由服务器即可访问整个集群，路由服务器内部会按照hash规则将数据存储到不同的`shard server`中。

## 扩容

扩容前需要确保新增的分片是没有任何操作的，即dbpath目录下没有任何文件。

### 新建dbpath和logpath

```shell
mkdir -p /usr/soft/mongo/mongodb/db/27014
mkdir -p /usr/soft/mongo/mongodb/logs/27014
```



### 启动新增的分片

```shell
./bin/mongod --shardsvr --replSet testSet1 --port 27014 --dbpath /usr/soft/mongo/mongodb/db/27014 --logpath /usr/soft/mongo/mongodb/logs/27014/log --fork --nojournal  
```

### 初始化新的分片

```shell
./bin/mongo --port 27014
#下面在mongo命令行中执行
config={_id:"testSet1", members:[{_id:0,host:"127.0.0.1:27014"}]}
rs.initiate(config);
```

### 添加新的分片到路由

```shell
./bin/mongo --port 27014
#下面在mongo命令行中执行
sh.addShard("testSet1/127.0.0.1:27014")
```

一个新的分片就添加到了现有的集群中。

## 建议

上面的过程只是实现的最简单的分片集群，但是并没有做高可用。

mongodb官方文档中建议，`config server`和`shard server`的每个分片都应该做3个以上的复制，这样保证在某一个节点发生故障时，整体服务仍然处于可以状态。复制集群的搭建参考`testSet`分片中端口27011和27012的初始化。

route server 也应当部署2个以上来保证高可用，在应用程序中按照连接多个mongodb的方式来连接多个路由服务器即可。

