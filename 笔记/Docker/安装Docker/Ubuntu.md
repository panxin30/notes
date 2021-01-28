### 参考文档
*   [Docker 官方 Ubuntu 安装文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
### OS requirements
### 查看ubuntu发行版本
```
cat /etc/issue 
cat /etc/os-release
```
To install Docker CE, you need the 64-bit version of one of these Ubuntu versions:
*   Cosmic 18.10
*   Bionic 18.04 (LTS)
*   Xenial 16.04 (LTS)

### Supported storage drivers
Docker CE on Ubuntu supports`overlay2`,`aufs`and`btrfs`storage drivers.
For new installations on version 4 and higher of the Linux kernel,`overlay2`is supported and preferred over`aufs`. Docker CE uses the`overlay2`storage driver by default. If you need to use`aufs`instead, you need to configure it manually. See[aufs](https://docs.docker.com/engine/userguide/storagedriver/aufs-driver/)
### Install using the repository
#### SET UP THE REPOSITORY
1.  Update the`apt`package index: 
    ~~~
    sudo apt-get update
    ~~~
2.  Install packages to allow`apt`to use a repository over HTTPS:
    ~~~
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    ~~~ 
3.  Add Docker’s official GPG key:
 ~~~
#国内源
sudo curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
#官方源
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 ~~~
然后，我们需要向`source.list`中添加 Docker 软件源

~~~shell
sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
# 官方源
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
~~~
> 以上命令会添加稳定版本的 Docker CE APT 镜像源，如果需要测试或每日构建版本的 Docker CE 请将 stable 改为 test 或者 nightly。
#### **安装 Docker CE**
更新 apt 软件包缓存，并安装`docker-ce`：
~~~
sudo apt-get update
sudo apt-get install docker-ce
~~~
#### **安装指定版本docker CE**
```
apt-cache madison docker-ce
sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu
```
### **docker 指定存储路径**
```
cat /etc/docker/daemon.json 

{

"graph": "/new-path/docker"

}
```
## **Uninstall Docker CE**

1.  Uninstall the Docker CE package:
    ~~~
    sudo apt-get purge docker-ce
    ~~~ 
2.  Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:
    ~~~
    sudo rm -rf /var/lib/docker
    ~~~