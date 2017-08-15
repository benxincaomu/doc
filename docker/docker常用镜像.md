

# 常用镜像

## nexus（maven私服）

maven私有仓库

### pull

```shell
docker pull sonatype/nexus
```

### run

```shell
docker run -d -p 8081:8081 --name nexus -v D:/docker/volume/nexus:/sonatype-work sonatype/nexus
```

### logs

```shell
docker logs -f nexus
```

### 中央仓库地址

```http
https://repo1.maven.org/maven2/
```

```http
http://maven.ibiblio.org/maven2/
```





## maven

### pull

```shell
docker pull maven
```

### run

```shell
docker run -it --rm --name maven -v "$PWD":/opt/project -v /root/.m2:/root/.m2 -w /opt/project maven mvn clean install
```

到`mvn`命令之前是指定maven的工作环境，`-v "$PWD":/opt/project`将当前目录挂载到容器中，`-w /opt/project`指定maven的工作目录为之前挂载的目录。

## mongodb

### pull

```shell
docker pull mongo
```

### run

```shell
docker run --name mongo -d -p 27017:27017 mongo
```



## tomcat

### pull

```shell
docker pull tomcat
```

### evn(7.0+)

```yaml
CATALINA_BASE:   /usr/local/tomcat
CATALINA_HOME:   /usr/local/tomcat
CATALINA_TMPDIR: /usr/local/tomcat/temp
JRE_HOME:        /usr
CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
```

### run

```shell
docker run -it --rm -v /yourWar.war:/usr/local/tomcat/webapps/sss.war -p 8888:8080 tomcat
```







## redis

### pull

```shell
docker pull redis
```

### run

```shell
docker run -p 6379:6379 --name redis -d redis redis-server --appendonly yes
```

### cli

```shell
docker run -it --link redis:redis --rm redis redis-cli -h redis -p 6379
```



## mysql

### pull

```shell
docker pull mysql
```

### run

映射主机3306端口访问mysql，`root`密码为`sftadmin`

```shell
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=sftadmin -d  mysql
```

#### 自定义配置文件

在主机`/my/custom`目录下建立`config-file.cnf`,并添加配置项，然后运行以下命令

```shell
docker run --name mysql -v /my/custom:/etc/mysql/conf.d -p 3306:3306  -e MYSQL_ROOT_PASSWORD=sftadmin -d  mysql
```

#### 自定义datadir

```shell
mkdir -p /my/own/datadir
docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql
```



#### 不使用配置文件

```shell
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=sftadmin -d  mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

查看命令行参数列表

```shell
docker run -it --rm mysql:tag --verbose --help
```

### cli

```shell

```



## gitlab(git服务器)

### pull

```shell
docker pull gitlab/gitlab-ce
```

### run

```shell
docker run -detach --hostname gitlab.sunft.com  -p 9001:80 -v D:/docker/volume/gitlab/config:/etc/gitlab -v D:/docker/volume/gitlab/logs:/var/log/gitlab -v D:/docker/volume/gitlab/data:/var/opt/gitlab  --name gitlab --restart always gitlab/gitlab-ce
```

挂在目录的时候总有错误，应该是windows下的权限问题，不挂载目录时可成功启动

```shell
docker run --detach --hostname gitlab.sunft.com  --publish 80:80 --name gitlab --restart always gitlab/gitlab-ce
```

 `--restart always` 使gitlab随着docker启动。 `host-name`需配置可访问的ip/域名，或者在`hosts`中配置映射，建议映射主机的80端口。



## sftp

### pull

```java
docker pull stilliard/pure-ftpd
```

### run

```shell
docker run -v D:/docker/volume/sftp/files/:/home/sunft/upload -p 33:22 -d atmoz/sftp sunft:sftadmin:::upload
```

## jenkins

### pull

```shell
docker pull jenkins
```

### run

```shell
docker run -d -p 8008:8080 --name jenkins -v D:/docker/volume/jenkins:/var/jenkins_home jenkins
```



## zookeeper

### pull

```shell
docker pull zoopeeker
```

### run

```shell
docker run --name zookeeper -p 2181:2181 -d zookeeper
```

自定义配置

```shell
docker run --name zookeeper --restart always -d -v $(pwd)/zoo.cfg:/conf/zoo.cfg zookeeper
```



## ssdb

### pull

```shell
docker pull felixsanz/ssdb
```

### run

```shell
docker run --rm -p 8888:8888 --name ssdb -d felixsanz/ssdb
```

自定义配置

`/usr/local/ssdb/ssdb.conf`是镜像中ssdb的配置文件路径，`/data`是ssbd的数据存储路径

```shell
docker run --rm -v /path/to/ssdb.conf:/usr/local/ssdb/ssdb.conf -v ssdb:/data -p 8888:8888 --name ssdb felixsanz/ssdb
```

### cli

环境准备

```shell
docker exec ssdb apk add python
```

连接

```shell
docker exec -it ssdb /usr/local/ssdb/ssdb-cli
```

## rabbitmq

### pull

```shell
docker pull rabbitmq
```

### run

```shell
docker run -d --hostname rabbit --name rabbit -p 15672:15672 rabbitmq:3-management
```

## ubuntu

### pull

```shell
docker pull ubuntu:16.04
```

### run

