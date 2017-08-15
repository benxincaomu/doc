# logstash简易部署文档

日志归集系统采用logstash接收各个系统输出的日志并发送到kafka，然后从kafka获取日志信息输出到elasticsearch做全文索引，由kibana做日志的展示

因为5.x版本的logstash中的log4j插件不能正常使用，所以需要使用2.4.0版本从接收日志信息。其他组件可使用官方最新的稳定版本。

## logstash

下载地址 https://www.elastic.co/downloads/past-releases/logstash-2-4-0

### 配置文件

在conf目录下新建ndsLog.conf，作为接收日志信息并输出到kafka的配置文件

```json
input{
   log4j{ //使用logstash的log4j插件，可使用多个，但是端口不能重复
       host =>"0.0.0.0" //使用本机ip，服务器有多个网卡时，建议绑定一个确定的ip
       port =>4560
       type =>"NotificationDeliveryScheduler"  //可作为项目名称区分
   }
}

output {
    stdout { codec => rubydebug }
    kafka{
        bootstrap_servers =>"192.168.10.229:9092"  //kafka的访问地址   
        topic_id =>"nds"  //topic
    }
}
```

在conf目录新建kafka-input.conf，作为从kafka收取数据并写入搜索引擎的配置文件

```json
input{
   kafka{
       zk_connect =>"192.168.10.229:2181"  //kafka连接的zookeeper地址
       topic_id =>"nds"
   }
}

output {
    stdout { codec => rubydebug }
    file {
        path => "/opt/data/java/logstash/logstash/logs/ndsLog-%{+YYYY-MM-dd}.log"
	   codec => plain 
    }
    elasticsearch { //写入搜索引擎
      hosts => ["192.168.10.229:9200"] 
      index => "logstash-%{+YYYY.MM.dd}"
    } 
}
```

**实际使用时需要删除上述配置文件中的注释方可使用**

### 启动

-f指定配置文件，-r可以实时获取配置文件的更新

```shell
开始接受应用日志: nohup ./bin/logstash -f conf/ndsLog.conf -r &
开始从kafka收取日志： nohup ./bin/logstash -f conf/kafka-input.conf -r &
```

### log4j接入logstash

```properties
log4j.logger.logstash=INFO
log4j.appender.logstash=org.apache.log4j.net.SocketAppender
#logstash所在ip
log4j.appender.logstash.RemoteHost=192.168.10.229
#logstash占用的端口
log4j.appender.logstash.Port=4560
log4j.appender.logstash.layout=org.apache.log4j.PatternLayout
log4j.appender.logstash.layout.ConversionPattern=%d [%-5p] [%l] %m%n
log4j.appender.logstash.ReconnectionDelay=10
log4j.appender.logstash.LocationInfo=true
```



## elasticsearch

下载地址 https://www.elastic.co/downloads/elasticsearch

### 配置

在`config/elasticsearch.yml `中加入`http.host : 0.0.0.0`,使其他机器可以访问elasticsearch，当服务器有多个网卡时，绑定其中一个ip即可，修改`http.port: 9200`可改变绑定的端口。

### 启动

```shell
nohup bin/elasticsearch &
```



## kibana

下载地址 https://www.elastic.co/downloads/kibana

### 配置

在`config/kibana.yml`文件中修改`server.port: 5601`来修改绑定的端口，修改`server.host: "0.0.0.0"`来绑定ip使其他ip可以访问。修改`elasticsearch.url : "http://localhost:9200"`来指定elasticsearch的访问地址。

### 启动

```shell
bin/kibana
```

### 界面配置

启动后访问kibana.yml中配置的地址和端口，会要求配置Index Patterns，可指定为 [logstash-]YYYY.MM.DD（因为在上述配置文件中指定的输出到elasticsearch的index名称为`logstash-%{+YYYY.MM.dd}`）。

之后可以使用`Discover`菜单来查询日志。



