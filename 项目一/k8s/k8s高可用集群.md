## 设置资源预留(未验证)
K8s 的 worker node 除了运行 pod 类进程外，还会运行很多其他的重要进程，包括 k8s 管理进程，如 kubelet、dockerd，以及系统进程，如 systemd。这些进程对整个集群的稳定性至关重要，因此需要为他们专门预留一定的资源。

节点拥有 32 核 CPU，64Gi 内存和 100Gi 存储。
为 k8s 管理进程预留了 1 核 CPU，2Gi 内存和 1Gi 存储。
为系统进程预留了 1 核 CPU，1Gi 内存和 1Gi 存储。
为内存资源设置了 500Mi 的驱逐阈值，为磁盘资源设置了 10% 的驱逐阈值。
在此场景下，节点可分配的 CPU 资源是 29 核，可分配的内存资源是 60.5Gi，可分配的磁盘资源是 88Gi。对于不可压缩资源，当 pod 的内存使用总量超过 60.5Gi 或者磁盘使用总量超过 88Gi 时，QoS 较低的 pod 将被优先驱逐。对于可压缩资源，如果节点上的所有进程都尽可能多的使用 CPU，则 pod 类进程加起来不会使用超过 29 核的 CPU 资源。

上述资源预留设置在 cluster.yml 中具体形式如下。
```
services:
  kubelet:
    extra_args:
      cgroups-per-qos: True
      cgroup-driver: cgroupfs
      kube-reserved: cpu=1,memory=2Gi,ephemeral-storage=1Gi
      kube-reserved-cgroup: /runtime.service
      system-reserved: cpu=1,memory=1Gi,ephemeral-storage=1Gi
      system-reserved-cgroup: /system.slice
      enforce-node-allocatable: pods,kube-reserved,system-reserved
      eviction-hard: memory.available<500Mi,nodefs.available<10%
```
关于资源预留更详细的内容可参考：https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/
------------------------------------------------------------------------------------------------------------------
# 一、节点规划
三主节点ubuntu20.04，一worker节点ubuntu20.04，阿里云内部SLB(172.31.0.28)做apiserver的负载均衡,
使用堆叠（stacked）控制平面节点，其中 etcd 节点与控制平面节点共存；
# 二、安装前的一些系统优化
内核参数/etc/sysctl.conf
```
net.core.somaxconn = 40480
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 4096 160777216
net.ipv4.tcp_wmem = 4096 4096 160777216
net.ipv4.tcp_mem = 786432 2097152 300145728
net.ipv4.tcp_max_syn_backlog = 46384
net.core.netdev_max_backlog = 50000
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_max_syn_backlog = 56384
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 131072
net.ipv4.ip_local_port_range = 1024 65535
net.netfilter.nf_conntrack_max = 2097152
fs.aio-max-nr = 524288
fs.file-max = 6590202
net.ipv4.ip_forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_timestamps = 0
```
----------------------------------------------------------------
/etc/security/limits.conf
```
# End of file
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```
---------------------------------------------------------------------------------------------
安装kubelet后kubelet优化
```
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --allowed-unsafe-sysctls=kernel.msg*,net.core.somaxconn,net.ipv4.tcp_keepalive_time,net.ipv4.tcp_syncookies,net.ipv4.tcp_tw_reuse,net.ipv4.tcp_timestamps,net.ipv4.tcp_fin_timeout
```
---------------------------------------------------------------------------------------------------------
k8s初始化后修改端口范围 会自动重启
cat /etc/kubernetes/manifests/kube-apiserver.yaml，新增参数
`- --service-node-port-range=80-65535`

因此,svc上的nodeport会监听在所有的节点上(如果不指定,即是随机端口,由apiserver指定--service-node-port-range '30000-32767'),即使有1个pod,任意访问某台的nodeport都可以访问到这个服务

# 三、 四台服务器都共同操作
## 允许iptables检查桥接流量
 ```
 cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
 ```
## 安装runtime(这里选择的docker，k8sv1.23后弃用docker)
 ```
 #
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
#
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
#
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
#
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
 ```
## 配置 Docker 守护程序，尤其是使用 systemd 来管理容器的 cgroup。
 ```
 sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1000m"
  },
  "storage-driver": "overlay2"
}
EOF
 ```
## 安装 kubeadm、kubelet 和 kubectl
```
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
#
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
#
sudo apt-mark hold kubelet kubeadm kubectl
```
# 安装网络插件
flannel 最简单，如果没有特别需求，这个就足够了，默认podcidr10.244.0.0/16
calico 除了提供网络还提供网络策略，默认podcidr192.168.0.0/16
192.168.0.1-192.168.255.254
# 四、高可用k8s集群配置
安装过程没什么好说的，照着官方文档做，主要是三台master服务器的hosts文件相关问题。
初始化命令`sudo kubeadm init --control-plane-endpoint "k8s-master-api:6443" --pod-network-cidr=10.244.0.0/16 --upload-certs`
```
root@k8s-master-01:~# cat /etc/hosts
172.31.0.32 k8s-master-01
172.31.0.31 k8s-master-02
172.31.0.30 k8s-master-03
172.31.0.32 k8s-master-api
root@k8s-master-02:~# cat /etc/hosts
172.31.0.32 k8s-master-01
172.31.0.31 k8s-master-02
172.31.0.30 k8s-master-03
172.31.0.31 k8s-master-api
root@k8s-master-03:~# cat /etc/hosts
172.31.0.32 k8s-master-01
172.31.0.31 k8s-master-02
172.31.0.30 k8s-master-03
172.31.0.30 k8s-master-api
root@k8s-slave-01:~# cat /etc/hosts
172.31.0.28 k8s-master-api
```
## 问题一、docker和kubelet的cgroupdriver要一致，都用的systemd

## 问题二、**kubeadm init 初始化的时候，需要访问k8s-master-api:6443，按理k8s-master-api在hosts文件中该写阿里云内部slb的IP地址(172.31.0.28)，但是k8s-master-01这台机器通过'172.31.0.28'访问不了自己6443端口**
修改为k8s-master-01内网IP后，初始化成功，但是后续如果有master节点当机的话，会是什么情况，需要做下测试。
**或者加haproxy**

## 问题三、master02和03的k8s-master-api改成他们各自内网IP的意义(还未修改，验证)

## 问题四、第一台master初始化完成后，安装网络插件成功后，master02和03才能加入集群

## master02和03加入集群
```
You can now join any number of the control-plane node running the following command on each as root:

kubeadm join k8s-master-api:6443 --token cjc4i6.43a52yzvgss6zgxr --discovery-token-ca-cert-hash sha256:2c031dc81826da97ff8f451a18594f9bbcfaf0d36d5cf5ff8374d9d18fe202bf --control-plane --certificate-key 2af4a703b3099044d059a0185582faad18531c96c68bc489052dd4ef6aabb3b4
```
# worker节点加入集群
```
kubeadm join k8s-master-api:6443 --token cjc4i6.43a52yzvgss6zgxr --discovery-token-ca-cert-hash sha256:2c031dc81826da97ff8f451a18594f9bbcfaf0d36d5cf5ff8374d9d18fe202bf 
```