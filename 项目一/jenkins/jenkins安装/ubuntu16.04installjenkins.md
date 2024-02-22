# 系统要求
最低推荐配置:
256MB可用内存
1GB可用磁盘空间(作为一个Docker容器运行jenkins的话推荐10GB)
为小团队推荐的硬件配置:
1GB+可用内存
50 GB+ 可用磁盘空间
# ubuntu 16.04 install jdk8
sudo apt-get update
sudo apt-get install openjdk-8-jdk
# ubuntu16.04 install jenkins
lwdevsrv01.firstgold.com
systemctl status gitlab-runsvdir
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt install jenkins=2.346.1
```
# 直接安装的jenkins最新版本要求JDK是11版的，而本次已经安装了JDK8,所以需要安装指定jenkins版本。
官网查询jenkins长期支持版本号
https://pkg.jenkins.io/debian/
# gitlab集合jenkins
# 使用gitlab-ctl reconfigure读取配置的时候，

nginx的主配置文件会恢复，也就是手动加的include会消失，没研究过有没有取消的方法

因此在gitlab.rb中停用nginx，手动安装nginx，再配置gitlab和jenkins
套用gitlab自带的nginx配置