## 在CentOS 6和CentOS 7上安裝PHP7X

### 安装yum源
```
#CentOS 7 只是启动方式不一样
sudo yum -y install epel-release
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
#CentOS 6
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
rpm -ivh epel-release-6-8.noarch.rpm
rpm -ivh remi-release-6.rpm
```
### 修改源
```
vim /etc/yum.repos.d/remi.repo
#将 [remi] 段中的 enabled=0 改为 enabled=1
vim /etc/yum.repos.d/remi-php71.repo #同时还有php72，php73在，看选择安装的是哪个版本，就改哪个
#将 [remi-php71] 段中的 enabled=0 改为 enabled=1
```
### 查看将要yum安装的php版本
```
yum list php
```
### 安装php及一些常用扩展
```
yum -y install php php-fpm php-cli php-pdo php-mysql php-gd php-bcmath php-xml php-mbstring php-mcrypt php-devel
```
### 命令
```
# 版本
[root@jmsite ~]# php -v
PHP 7.1.25 (cli) (built: Dec  8 2018 14:07:15) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies......
# 扩展
php -m
#php-fpm命令
# 启动
service php-fpm start
# 重启
service php-fpm restart
# 停止
service php-fpm stop
#设置开机启动
chkconfig php-fpm on
```
# ubuntu 安装php
```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt list | grep php
#选择要安装的版本7.1,7.2,7.3......
```