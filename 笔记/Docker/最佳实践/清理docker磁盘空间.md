`docker system df`命令，类似于Linux上的df命令，用于查看Docker的磁盘使用情况
docker system prune命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)。`docker system prune -a`命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚吶。

### 限制容器日志大小(未经实验)
在Ubuntu上，Docker的所有相关文件，包括镜像、容器等都保存在/var/lib/docker/目录中
```
nginx:
  image: nginx:1.12.1
  restart: always
  logging:
    driver: "json-file"
    options:
      max-size: "5g"
```