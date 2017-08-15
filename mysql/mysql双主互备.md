# Mysql双主互备

## 配置文件

主1配置

```cnf
[mysqld]
basedir =/usr/local/mysql
datadir =/usr/local/mysql/data
port = 3206
server_id = 1
log-bin= 3206-bin
binlog_format = mixed

read-only=0
#binlog-do-db=test
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
auto-increment-offset=1
auto-increment-increment=2
```

主2配置

```cnf
[mysqld]
basedir =/usr/local/mysql
datadir =/usr/local/mysql/data
port = 3206
server_id = 2
log-bin= 3207-bin
binlog_format = mixed

read-only=0
#binlog-do-db=test
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
auto-increment-offset=2
auto-increment-increment=2
```

### 配置说明

```yaml
说明: 
log-bin : 要启用二进制日志
server_id : 用于标识不同的数据库服务器

binlog-do-db : 需要记录到二进制日志的数据库
binlog-ignore-db : 忽略记录二进制日志的数据库
auto-increment-offset : 该服务器自增列的初始值。
auto-increment-increment : 该服务器自增列增量。

replicate-do-db : 指定复制的数据库
replicate-ignore-db : 不复制的数据库
relay_log : 从库的中继日志，主库日志写到中继日志，中继日志再重做到从库。
log-slave-updates : 该从库是否写入二进制日志，如果需要成为多主则可启用。只读可以不需要。
```

## 命令行操作

### 创建同步用户

主1和主2创建同步用户，用于对方同步数据

```shell
grant replication slave on *.* to 'sunft'@'%' identified by 'sunftadmin'; 
FLUSH PRIVILEGES;
```

这里是所有远程主机都可以同步，可根据具体需求指定需要同步的database、table以及同步用户的访问ip、用户名及密码

分别在主1和主2执行`show master status`,并记录下File和position的值

我的主1的File是`3206-bin.000001`,position是327

我的主2的File是`3207-bin.000001`,position是327

在主1上使用主2创建的同步用户登录主2，验证用户和授权。在主2上以相同方式验证主1的用户和授权。

### 配置同步

主1同步主2配置

```shell
change master  to master_host='192.168.112.134',master_port=3207,master_user='sunft',master_password='sunftadmin',master_log_file='3207-bin.000001',master_log_pos=327;
start slave;
```

其中master_log_file和master_log_pos参数就是刚刚在主1上查看binlog信息对应值 ，在主2上也是如此

主2同步主1配置

```shell
change master  to master_host='192.168.112.134',master_port=3206,master_user='sunft',master_password='sunftadmin',master_log_file='3206-bin.000001',master_log_pos=327;
start slave;
```

查看同步状态

```shell
show slave status;
```

如果`Slave_IO_State='Waiting for master to send event'`,`Slave_IO_Running=Yes`,`Slave_SQL_Running=Yes` 主从复制搭建成功。两台主机主从都验证成功，则双主互备搭建成功。

如有异常，确认主1与主2之间的网络通畅，然后重启同步服务进行重试

```shell
stop slave;
start slave;
```

或者根据同步状态查找相关解决方案。



