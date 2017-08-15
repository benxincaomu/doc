# docker安装手册

## centos下安装

```shell
yum remove docker docker-common container-selinux docker-selinux docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-edge #可选
yum-config-manager --enable docker-ce-testing #可选
yum-config-manager --disable docker-ce-edge #可选
yum makecache fast
yum install -y docker-ce
```

