参考：https://linux.cn/article-5926-1.html 
参考：https://wiki.archlinux.org/index.php/Systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
在Linux生态系统中，Systemd被部署到了大多数的标准Linux发行版中，只有为数不多的几个发行版尚未部署。Systemd通常是所有其它守护进程的父进程，但并非总是如此。

# **管理单个 unit**
**systemctl \[command\] \[unit\]**  
command 主要有：  
**start**：立刻启动后面接的 unit。  
**stop**：立刻关闭后面接的 unit。  
**restart**：立刻关闭后启动后面接的 unit，亦即执行 stop 再 start 的意思。  
**reload**：不关闭 unit 的情况下，重新载入配置文件，让设置生效。  
**enable**：设置下次开机时，后面接的 unit 会被启动。  
**disable**：设置下次开机时，后面接的 unit 不会被启动。  
**status**：目前后面接的这个 unit 的状态，会列出有没有正在执行、开机时是否启动等信息。  
**is-active**：目前有没有正在运行中。  
**is-enable**：开机时有没有默认要启用这个 unit。  
**kill**：不要被 kill 这个名字吓着了，它其实是向运行 unit 的进程发送信号。  
**show**：列出 unit 的配置。  
**mask**：注销 unit，注销后你就无法启动这个 unit 了。  
**unmask**：取消对 unit 的注销。
### **kubelet.service的基本信息**
`systemctl status kubelet`
```
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Sat 2020-06-20 07:16:59 CST; 1 months 0 days ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 3507 (kubelet)
    Tasks: 28 (limit: 4915)
   CGroup: /system.slice/kubelet.service
           └─3507 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1 --resolv-conf=/run/systemd/resolve/resolv.conf --allowed-unsafe-sysctls=kernel.msg*,net.core.somaxconn,net.ipv4.tcp_keepalive_time,net.ipv4.tcp_syncookies,net.ipv4.tcp_tw_reuse,net.ipv4.tcp_timestamps,net.ipv4.tcp_fin_timeout
```
输出内容的第一行是对 unit 的基本描述。  
 **Loaded** 描述操作系统启动时会不会启动这个服务，enabled 表示开机时启动，disabled 表示开机时不启动。而启动该服务的配置文件路径为：/lib/systemd/system/kubelet.service。
**Drop-In**自定义参数和环境变量，使用systemd的drop-in文件，会覆盖默认的/lib/systemd/system/kubelet.service文件
**Tasks**
**CGroup**属于哪个控制群组
# **查看系统上的 unit**
查看已启动的服务列表：`systemctl list-unit-files|grep enabled`
 查看已启动的服务列表：`systemctl list-unit-files --type service`
查看启动失败的服务列表：`systemctl --failed`

# **检查 unit 之间的依赖性**
```
root@cn-office-saas-test:~# systemctl get-default
graphical.target
root@cn-office-saas-test:~# systemctl list-dependencies default.target 
default.target
● ├─accounts-daemon.service
● ├─apport.service
● ├─display-manager.service
● ├─grub-common.service
● ├─systemd-update-utmp-runlevel.service
● ├─ureadahead.service
● └─multi-user.target
●   ├─apport.service
●   ├─atd.service
●   ├─console-setup.service
●   ├─cron.service
●   ├─dbus.service
```
--reverse 选项查看 multi-user.target unit 被谁使用：

`systemctl list-dependencies multi-user.target --reverse`

# **相关的目录和文件**
在不同的发行版中与 systemd 相关的文件路径可能会不太一样，强调一下，本文介绍的是 ubuntu 16.04 。
/lib/systemd/system/ 大多数 unit 的配置文件都放在这个目录下。
/run/systemd/system/ 系统运行过程中产生的脚本，比如用户相关的脚本和会话相关的脚本。
/etc/systemd/system/ 这个目录中主要的文件都是指向 /lib/systemd/system/ 目录中的链接文件。
注意，在我们自己创建 unit 配置文件时，既可以把配置文件放在 /lib/systemd/system/ 目录下，也可以放在 /etc/systemd/system/ 目录下。
/etc/default/ 这个目录中放置很多服务默认的配置文件。
/var/lib/ 一些会产生数据的服务都会将他的数据写入到 /var/lib/ 目录中，比如 docker 相关的数据文件就放在这个目录下。
/run/  这个目录放置了好多服务运行时的临时数据，比如 lock file 以及 PID file 等等。

我们知道 systemd 里管理了很多会用到本机 socket 的服务，所以系统中肯定会产生很多的 socket 文件。那么，这些 socke 文件都存放在哪里呢？我们可以使用 systemctl 进行查看：
`systemctl list-sockets`

# **systemctl daemon-reload 子命令**
*   新添加 unit 配置文件时需要执行 daemon-reload 子命令
*   有 unit 的配置文件发生变化时也需要执行 daemon-reload 子命令

# **Cgroups 与 Systemd**
Systemd 是一个强大的 init 系统，它甚至为我们使用 cgorups 提供了便利！Systemd 提供的内在机制、默认设置和相关的操控命令降低了配置和使用 cgroups 的难度，即便是 Linux 新手，也能轻松的使用 cgroups 了。
参考：https://www.cnblogs.com/sparkdev/p/9523194.html
参考：https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html-single/resource_management_guide/index

# **什么是控制群组(cgroup)**
参考：https://www.cnblogs.com/sparkdev/p/8296063.html
参考：https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html-single/resource_management_guide/index#sec-What_are_Control_Groups
*控制群组（control group）*（简写为*cgroup*）是 Linux kernel 的一项功能：在一个系统中运行的层级制进程组，您可对其进行资源分配（如 CPU 时间、系统内存、网络带宽或者这些资源的组合）。通过使用 cgroup，系统管理员在分配、排序、拒绝、管理和监控系统资源等方面，可以进行精细化控制。硬件资源可以在应用程序和用户间智能分配，从而增加整体效率。
### **cgroups 的主要作用**

实现 cgroups 的主要目的是为不同用户层面的资源管理提供一个统一化的接口。从单个任务的资源控制到操作系统层面的虚拟化，cgroups 提供了四大功能：

*   资源限制：cgroups 可以对任务是要的资源总额进行限制。比如设定任务运行时使用的内存上限，一旦超出就发 OOM。
*   优先级分配：通过分配的 CPU 时间片数量和磁盘 IO 带宽，实际上就等同于控制了任务运行的优先级。
*   资源统计：cgoups 可以统计系统的资源使用量，比如 CPU 使用时长、内存用量等。这个功能非常适合当前云端产品按使用量计费的方式。
*   任务控制：cgroups 可以对任务执行挂起、恢复等操作。

## **设置java服务**
```
vim /etc/systemd/system/bw-custom.service
[Unit]
Description=bw-custom
After=supervisord.service

[Service]
WorkingDirectory=/home/brokerwork/bw-custom
ExecStart=/usr/bin/java -server -Xms1024M -Xmx1024M -XX:+UseNUMA -XX:+UseParallelGC -XX:NewRatio=1 -XX:MaxDirectMemorySize=512M -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=dump/ -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -verbosegc -Xloggc:dump/gc.log -Dlogging.config=classpath:logback-deploy.xml -jar bw-custom.jar --spring.profiles.active=preprod

SuccessExitStatus=143

User=brokerwork
Group=brokerwork

[Install]
WantedBy=multi-user.target
```








