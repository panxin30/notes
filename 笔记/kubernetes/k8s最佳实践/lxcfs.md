Linuxs利用Cgroup实现了对容器的资源限制，但在容器内部依然缺省挂载了宿主机上的`procfs`的`/proc`目录，其包含如：meminfo, cpuinfo，stat， uptime等资源信息。一些监控工具如free/top或遗留应用还依赖上述文件内容获取资源配置和使用情况。当它们在容器中运行时，就会把宿主机的资源状态读取出来，引起错误和不便。

### LXCFS简介

社区中常见的做法是利用[lxcfs](https://yq.aliyun.com/go/articleRenderRedirect?url=https://github.com/lxc/lxcfs)来提供容器中的资源可见性。lxcfs 是一个开源的FUSE（用户态文件系统）实现来支持LXC容器，它也可以支持Docker容器。

LXCFS通过用户态文件系统，在容器中提供下列`procfs`的文件。
## Install lxcfs
```
#ubuntu 16.04
sudo apt-get update
sudo apt-get install lxcfs
```