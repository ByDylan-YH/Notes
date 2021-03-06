# 下载地址
- https://mirrors.aliyun.com/centos/

## 修改网卡配置
```sh
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

# centos 7
systemctl restart network
# centos 8
systemctl restart NetworkManager
ip addr
ipconfig
```
### 网络地址转换(NAT)模式
```sh
ONBOOT=yes
```
### 桥接网卡模式
```sh
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.1.201
NETMASK=255.255.255.0
GETEWAY=192.168.1.1
DNS1=192.168.1.1
```

## 修改主机名 Hostname
```sh
vi /etc/hostname
by201

vim /etc/hosts
192.168.1.201 by201
192.168.1.202 by202
192.168.1.203 by203

vim /etc/sysconfig/network
hostname=by201
```

## ssh免密码登录配置
```sh
vi /etc/hosts

192.168.1.201 by201
192.168.1.202 by202
192.168.1.203 by203

ssh-keygen -t rsa  
cd /root/.ssh  id_rsa(私钥) id_rsa.pub(公钥)

ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.201
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.202
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.203
```

## Centos7修改repo源配置
- http://mirrors.aliyun.com/repo/Centos-7.repo
- http://mirrors.163.com/.help/CentOS7-Base-163.repo

## Centos8修改repo源配置
```sh
cd /etc/yum.repos.d/

[BaseOS]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
##additional packages that may be useful
[Extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
##additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/os/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
[PowerTools]
name=CentOS-$releasever - PowerTools - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/PowerTools/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/PowerTools/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/PowerTools/$basearch/os/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official

[AppStream]
name=CentOS-$releasever - AppStream - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/AppStream/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/AppStream/$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
```

## 测试
```sh
sudo dhclient
yum -y update
yum install -y vim net-tools lsof tree npm nodejs git zip mlocate httpd createrepo iptables iptables-services tar chrony bzip2 
```

## 防火墙,iptables安装
- https://www.cnblogs.com/kreo/p/4368811.html

### 1.安装 iptables
```sh
yum install -y iptables iptables-services
```
### 2.禁用 firewalld
```sh
systemctl status firewalld
systemctl stop firewalld
systemctl mask firewalld
```

### 选择性禁用
```sh
# 注意看下是不是软连接 -> /etc/sysconfig/selinux
ll /etc/sysconfig/selinux

SELINUX=disabled

# 重启后验证
sestatus
```

### 3.设置开放端口,必须写在黄色上面
```sh
vim /etc/sysconfig/iptables 
-A INPUT -p tcp --dport 21 -j ACCEPT
-A INPUT -p tcp --dport 22 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 3306 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```
### 4.启用 iptables
```
service iptables save
systemctl enable iptables.service
systemctl start iptables.service

systemctl disable iptables.service
systemctl mask iptables.service
```

## 创建本地云源repo压缩包
### 1.上传Centos镜像
### 2.挂载到一个目录
```sh
mkdir /iso
mount Centos** /iso
```
### 3.进入目录
```
cd /iso/AppStream/Packages
createrepo /var/www/html/openstack
```

## 一些报错信息
### 配置Windows ftp linux 报错:200 PORT command successful. Consider using PASV.425 Failed to establish connection.
```sh
vim /etc/vsftpd/vsftpd.conf
pasv_enable=YES
pasv_min_port=6000
pasv_max_port=7000

vim /etc/sysconfig/iptables 
-A INPUT -p tcp --dport 6000:7000 -j ACCEPT
  
vim /etc/selinux/config
SELINUX=disabled
```
### ping baidu.com 出现 connect: network is unreachable
```sh
sudo dhclient
sudo systemctl stop dhcpcd
sudo systemctl start dhcpcd
```
### 所有命令都无法使用
```sh
export PATH=/bin:/usr/bin:$PATH
source ~/.bash_profile 
```

## Centos 8 过期的一些包
### unzip 改为 zip
### ntpdate 改为 chrony

## 一些报错信息
### 解除挂载报错:target is busy
```sh
yum install psmisc
fuser -mv /iso ## 把下面的进程kill掉
```
### Virtualbox未关机就关闭电脑,无法启动,unmount and run xfs_repair
```sh
ls -l /dev/mapper
xfs_repair /dev/mapper/cl_muban-root
# 如果报错 The filesystem has valuable metadata change in a log ...
xfs_repair -L /dev/mapper/cl_muban-root
```

## 一些安装过程
### 安装cgconfig服务
```sh
yum install -y libcgroup libcgroup-tools
# 检查合法性
cgconfigparser -l /etc/cgconfig.conf
# 启动
systemctl start cgconfig.service
```