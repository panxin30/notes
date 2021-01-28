[https://yeasy.gitbooks.io/docker\_practice/](https://yeasy.gitbooks.io/docker_practice/)
**Docker参考手册**:
https://docs.docker.com/engine/reference/commandline/dockerd/

默认docker是没有资源限制的。

新版docker将命令进行了归类，以前的命令格式还能用，但会逐渐取消。
```
Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  engine      Manage the docker engine
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes
```
```
# docker --help
# docker  version #查看版本
# docker  search centos#搜索可用docker镜像
# docker  images 查看当前docker所有镜像
# docker  pull  centos #下载镜像
# docker history 镜像 # 可以查看这个镜像每层大小

#####根据容器id导入导出
# cat  centos.tar | docker import -  centos6  #Docker导入镜像
# dockerexport  id > cenos6.tar  #Docker导出镜像，这个id是容器id不是镜像id
#####根据镜像id导入导出
# docker save 0fdf2b4c26d3 > centos6.tar
# docker load <  centos6.tar

# docker  run   centos echo "hello word"#在docker容器中运行hello world!
# docker  run  centos yum install ntpdate#在容器中安装ntpdate的程序
# docker  ps -l 命令获得最后一个容器的id，docker   ps  -a查看所有的容器。
# 运行docker commit 提交刚修改的容器，例如：
# docker commit 2313132  centos:v1

#  docker run -i -t centos /bin/bash 
# docker run -it mysql:5.7.26 /bin/bash 镜像加tag
# 在容器里启动一个/bin/bash shell环境，可以登录进入操作，其中-t 表示打开一个终端的意思，-i表示可以交互输入。

# docker run  -d centos:v1  /bin/bash  ,-d表示在后台启动，以daemon方式启动。  
# docker stop  id 关闭容器
# docker start  id 启动某个容器
# docker rm  id 删除容器，docker  rmi  images删除镜像

# docker run  -d -p 80:80  -p 8022:22  centos:v2
#解析：-p指定容器启动后docker上运行的端口映射及容器里运行的端口，80:80，第一个80表示docker系统上的80，第二个80表示docker虚拟机里面的端口。用户默认访问本机80端口，自动映射到容器里面的80端口。
# docker  exec   -it  id  /bin/bash
docker tag 120.76.0.0:8082/alpine-jdk8:v1 192.168.0.96:8082/alpine-jdk8:v1
```