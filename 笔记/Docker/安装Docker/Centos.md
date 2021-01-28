### 参考文档
*   [Docker 官方 CentOS 安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)。
## Get Docker CE for CentOS
### OS requirements
To install Docker CE, you need a maintained version of **CentOS 7**. Archived versions aren’t supported or tested.
The`centos-extras`repository must be enabled. This repository is enabled by default, but if you have disabled it, you need to[re-enable it](https://wiki.centos.org/AdditionalResources/Repositories).
The`overlay2`storage driver is recommended. recommended推荐
Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如`overlay2`存储层驱动）无法使用，并且部分功能可能不太稳定。
### Uninstall old versions
Older versions of Docker were called`docker`or`docker-engine`. If these are installed, uninstall them, along with associated dependencies.
~~~
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
~~~
### Install using the repository

Before you install Docker CE for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

#### SET UP THE REPOSITORY

1.  Install required packages.`yum-utils`provides the`yum-config-manager`utility, and`device-mapper-persistent-data`and`lvm2`are required by the`devicemapper`storage driver.
    
    ~~~
    sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
    
    ~~~
    
2.  Use the following command to set up the **stable** repository.
    
    ~~~
    #国内源
    sudo yum-config-manager \
        --add-repo \
        https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
    # 官方源
    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    ~~~
#### INSTALL DOCKER CE
~~~
sudo yum makecache fast
sudo yum install docker-ce
~~~

1.  Install the*latest version*of Docker CE and containerd, or go to the next step to install a specific version:
    ~~~
    sudo yum install docker-ce docker-ce-cli containerd.io
    ~~~
To install a*specific version*of Docker CE, list the available versions in the repo, then select and install:
~~~
yum list docker-ce --showduplicates | sort -r
~~~
### 添加内核参数

如果在 CentOS 使用 Docker CE 看到下面的这些警告信息：

~~~
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
~~~

请添加内核配置参数以启用这些功能。

~~~
sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

~~~

然后重新加载`sysctl.conf`即可

~~~
sudo sysctl -p
~~~
## **Uninstall Docker CE**

1.  Uninstall the Docker package:
    
    ~~~
    $ sudo yum remove docker-ce
    ~~~
    
2.  Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:
    
    ~~~
    sudo rm -rf /var/lib/docker
    ~~~