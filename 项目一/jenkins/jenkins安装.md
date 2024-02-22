# 系统要求
最低推荐配置:
256MB可用内存
1GB可用磁盘空间(作为一个Docker容器运行jenkins的话推荐10GB)
为小团队推荐的硬件配置:
1GB+可用内存
50 GB+ 可用磁盘空间
# ubuntu 20.04 jenkins安装
参考:https://www.jenkins.io/doc/book/installing/
参考:https://www.linuxtechi.com/install-configure-jenkins-ubuntu-20-04/
# 一、安装前提
Java 8 or Java 11 are required for running modern versions of Jenkins. All other Java versions are not supported.
# 二、安装JDK
如果没安装会有提示
```
root@cn-office-tonytest-jenkins:/home/ubuntu# java -version

Command 'java' not found, but can be installed with:
apt install openjdk-11-jre-headless  # version 11.0.13+8-0ubuntu1~20.04
apt install openjdk-8-jre-headless   # version 8u312-b07-0ubuntu1~20.04
```

```
root@ali-hn-lw-jenkins-new:~# cat /var/lib/jenkins/config.xml | grep version
  <version>2.190.2</version>
dpkg -l | grep jenkins  
```
# 三、安装jenkins
apt install jenkins
或
apt install jenkins=2.190.2


