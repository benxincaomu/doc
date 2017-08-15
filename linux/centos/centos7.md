# centos7

## 关闭系统防火墙

```shell
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```



## 服务自启

```shell
chkconfig serviceName on ##服务自启
chkconfig serviceName off  ##服务不自启
```

## 图形界面

### 安装

```shell
sudo  yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
```

### 设置默认

命令设置

```shell
sudo ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target  ##默认访问图形界面
sudo ln -sf /lib/systemd/system/runlevel3.target /etc/systemd/system/default.target  ##默认访问命令行界面
```





## 常用软件

### gitlab

```shell
yum install curl policycoreutils openssh-server openssh-clients
systemctl enable sshd
systemctl start sshd
yum install postfix
systemctl enable postfix
systemctl start postfix
firewall-cmd --permanent --add-service=http
systemctl reload firewalld

curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | sudo bash
gitlab-ctl reconfigure #启动
```

