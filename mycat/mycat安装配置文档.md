# mycat简单配置说明

## 1 目录结构

### 1.1 目录说明

​	bin:相关脚本文件目录

​	conf:配置文件目录

​	lib:依赖jar包目录

​	logs:日志输出目录



## 2 单点启动配置

单点启动依赖于xml文件进行配置

### 2.1 schema.xml配置

#### 2.1.1 dataHost标签

​	该标签在mycat逻辑库中也是作为最底层的标签存在，直接定义了具体的数据库实例、读写分离配置和心跳语句。

	##### 2.1.1.1 标签属性

​	

| 属性名       | 用途                                       | 值类型     |
| --------- | ---------------------------------------- | ------- |
| name      | 唯一标识dataHost标签，供上层的标签使用。                 | String  |
| maxCon    | 每个读写实例连接池的最大连接数量                         | Integer |
| minCon    | 每个读写实例连接池的最小连接数量                         | Integer |
| balance   | 读负载均衡类型：0--不进行负载均衡，1--所有数据库实例都进行select语句的负载，2--所有读操作都随机的在writeHost、readhost上分发，3--所有读请求随机的分发到wiriterHost对应的readhost执行，writerHost不负担读压力 | Integer |
| writeType | 写负载均衡类型：0--所有写操作发送到配置的第一个writeHost，第一个挂了切到还生存的第二个writeHost | Integer |
| dbType    | 后端连接的数据库类型：mysql、mongodb、oracle、spark    | String  |
| dbDriver  | 连接后端数据库使用的Driver，目前可选的值有native和JDBC      | String  |





#### 2.1.2 子标签





