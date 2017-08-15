`sudo vim /etc/network/interfaces`

添加一行`dns-nameservers 114.114.114.114`

`sudo resolvconf -u`

`cat /etc/resolv.conf` 

如最后输出的文件留包含114DNS，则成功