# docker常用命令

## 查看docker版本

```shell
docker  version
```

## 查看docker系统信息

```docker
docker info
```



## 镜像



### 检索镜像

```shell
docker search IMAGE
```



### 已安装镜像

```
docker images
```



### 下载镜像

```shell
docker pull IMAGE
```



### 卸载镜像

```
docker rmi IMAGE[:TAG]
```



### 启动

```shell
docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]
```

启动镜像后会有一个containerID ,如果参数中指定，容器还会有name 

#### 常用参数

```shell
-d  #后台运行
-i  #以交互模式运行容器，通常与 -t 同时使用
-t  #为容器重新分配一个伪输入终端，通常与 -i 同时使用
-e  #为容器指定环境变量
--name containerName  #为容器指定一个名称
--expose=[] #开放一个端口或一组端口
-p 80:8080 #端口映射，通过主机80端口访问容器的8080端口
-v /localPath:/dockerPath #将本机目录或文件挂在到container下的指定目录或文件
--rm #运行结束后删除容器
--link [] #链接到其他容器
-w     #指定工作目录
--expose=[]  ##开放一组端口

###  以下是内存分配的相关参数
-m   #内存限制，格式是数字加单位，单位可以为 b,k,m,g。最小为 4M
--memory #同 -m
--memory-swap #	内存+交换分区大小总限制。格式同上。必须必-m设置的大
--memory-reservation	#内存的软性限制。格式同上
--oom-kill-disable	#是否阻止 OOM killer 杀死容器，默认没设置
--oom-score-adj	   #容器被 OOM killer 杀死的优先级，范围是[-1000, 1000]，默认为 0
--memory-swappiness	 #用于设置容器的虚拟内存控制行为。值为 0~100 之间的整数
--kernel-memory	 #核心内存限制。格式同上，最小为 4M

### CPU限制的相关参数
--cpuset-cpus=""	#允许使用的 CPU 集，值可以为 0-3,0,1
-c,--cpu-shares=0	#CPU 共享权值（相对权重）
--cpu-period=0	    #限制 CPU CFS 的周期，范围从 100ms~1s，即[1000, 1000000]
--cpu-quota=0	    #限制 CPU CFS 配额，必须不小于1ms，即 >= 1000
--cpuset-mems=""	#允许在上执行的内存节点（MEMs），只对 NUMA 系统有效
```



### build

```shell
docker build -t IMAGE[:TAG] .
```

#### Dockerfile

```dockerfile
FROM java   #基础镜像
MAINTAINER sunft #维护人员及联系方式
ENV JAVA_HOME=/opt/java  #设置环境变量
WORKDIR /usr/  #工作目录，可多次使用，相当于  RUN cd /usr/
COPY aa.txt /opt/  #复制本地文件到镜像中的指定路径
RUN yum install vim -y  #构建过程中执行的命令
EXPOSE 8080 #镜像开放的端口，可在多行配置多个端口
CMD echo "aaa" #启动时执行的命令，但是一个Dockerfile中只能有一条CMD命令，多条则只执行最后一条CMD
USER root  #镜像的执行用户
```



## 容器

### 启动容器

```shell
docker start containerName
docker start containerID
```



### 停止容器

```
docker kill containerID
docker kill containerName

docker stop containerID
docker stop containerName
```

kill为强制退出命令

### 删除容器

只可以删除已停止的容器

```shell
docker rm containerID
docker rm containerName
```

## 异常操作

### 删除tag为none的images

```shell
docker images|grep none|awk '{print $3}'|xargs docker rmi
```



