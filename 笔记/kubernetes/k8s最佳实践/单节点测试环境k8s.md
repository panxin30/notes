环境：ubuntu 18.04,k8s 1.16.1,docker 18.09 数据盘使用xfs格式
一、服务器优化
cat /etc/sysctl.conf
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
二、kubelet优化
```
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
```
root@saas-test:~/k8s-yaml/crm-sprod# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --allowed-unsafe-sysctls=kernel.msg*,net.core.somaxconn,net.ipv4.tcp_keepalive_time,net.ipv4.tcp_syncookies,net.ipv4.tcp_tw_reuse,net.ipv4.tcp_timestamps,net.ipv4.tcp_fin_timeout
```
### 修改端口范围 会自动重启
cat /etc/kubernetes/manifests/kube-apiserver.yaml
`- --service-node-port-range=80-65535`

三、
### 
因此,svc上的nodeport会监听在所有的节点上(如果不指定,即是随机端口,由apiserver指定--service-node-port-range '30000-32767'),即使有1个pod,任意访问某台的nodeport都可以访问到这个服务
### 
externalIPs 通过svc创建,在指定的node上监听端口