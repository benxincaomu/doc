# docker私服搭建

## 代理服务器搭建

### pull

```shell
docker pull registry
```

### run

```shell
docker run -d -v D:/docker/volume/registry/config.yml:/etc/docker/registry/config.yml -p 5000:5000  --restart always --name registry registry
```

## 测试

### 客户机

#### 配置

linux下默认配置文件为`/etc/docker/daemon.json`,如果本机无此文件需要手动新建，并才文件中加入下列配置（本机ip为10.3.21.20,registry端口为5000）

```json
{
    "insecure-registries": [
        "10.3.21.20:5000"
    ]
}
```

#### 创键本地镜像

```shell
docker pull centos
docher tag centos 10.3.21.20:5000/localos
```

#### push

```shell
docher push 10.3.21.20:5000/localos
```

push成功后会有以下信息输出

```
The push refers to a repository [10.3.21.20:5000/localos]
dc1e2dcdc7b6: Mounted from centos
```

#### 查看本地仓库镜像

```shell
[root@bogon ~]# curl http://10.3.21.20:5000/v2/_catalog
```

输出`{"repositories":["localos"]}`，数组中则为本地仓库中的镜像

#### 删除本地库中的镜像（暂未成功）

```shell
curl -I -X DELETE 10.3.21.20:5000/v2/test/manifests/sha256:956ad85bb6be3837c8cf31cc3bc0edb9be9599296068dc85922eaf18bc313054
```









