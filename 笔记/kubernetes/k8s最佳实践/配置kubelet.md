参考：https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/kubelet-integration/
参考：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/
# 简介
kubelet 是在每个 Node 节点上运行的主要 “节点代理”。
kubelet 是基于 PodSpec 来工作的。每个 PodSpec 是一个描述 Pod 的 YAML 或 JSON 对象。kubelet 接受通过各种机制（主要是通过 apiserver）提供的一组 PodSpec，并确保这些 PodSpec 中描述的容器处于运行状态且运行状况良好。kubelet 不管理不是由 Kubernetes 创建的容器。
**除了来自 apiserver 的 PodSpec 之外，还可以通过以下三种方式将容器清单（manifest）提供给 kubelet。**
File（文件）：利用命令行参数给定路径。kubelet 周期性地监视此路径下的文件是否有更新。监视周期默认为 20s，且可通过参数进行配置。

# 系统中的 kubelet 插件 
kubeadm 中附带了有关系统如何运行 kubelet 的配置。 请注意 kubeadm CLI 命令不会触及此插件。
通过 kubeadm DEB 或者 RPM 包 安装的配置文件已被写入 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 并由系统使用。 它加强了基础设施 kubelet.service for RPM (resp. kubelet.service for DEB))：
```
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# 这是 "kubeadm init" 和 "kubeadm join" 运行时生成的文件，动态地填充 KUBELET_KUBEADM_ARGS 变量
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# 这是一个文件，用户在不得已下可以将其用作替代 kubelet args。
# 用户最好使用 .NodeRegistration.KubeletExtraArgs 对象在配置文件中替代。
# KUBELET_EXTRA_ARGS 应该从此文件中获取。
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```
该文件为 kubelet 指定由 kubeadm 管理的所有文件的默认位置。

* 用于 TLS 引导程序的 KubeConfig 文件为 /etc/kubernetes/bootstrap-kubelet.conf， 但仅当/etc/kubernetes/kubelet.conf 不存在时才能使用。
* 具有唯一 kubelet 标识的 KubeConfig 文件为 /etc/kubernetes/kubelet.conf。
* 包含 kubelet 的组件配置的文件为 /var/lib/kubelet/config.yaml。
* 包含的动态环境的文件 KUBELET_KUBEADM_ARGS 是来源于 /var/lib/kubelet/kubeadm-flags.env。
* 包含用户指定标志替代的文件 KUBELET_EXTRA_ARGS 是来源于 /etc/default/kubelet（对于 DEB），或者 /etc/sysconfig/kubelet（对于 RPM）。 KUBELET_EXTRA_ARGS 在标志链中排在最后，并且在设置冲突时具有最高优先级。
# **为系统守护进程预留计算资源** kubelet
参考：https://kubernetes.io/zh/docs/tasks/administer-cluster/reserve-compute-resources/
### **示例场景** 
这是一个用于说明节点分配计算方式的示例：
```
节点拥有 32Gi 内存，16 核 CPU 和 100Gi 存储
--kube-reserved 设置为 cpu=1,memory=2Gi,ephemeral-storage=1Gi
--system-reserved 设置为 cpu=500m,memory=1Gi,ephemeral-storage=1Gi
--eviction-hard 设置为 memory.available<500Mi,nodefs.available<10%
```
在这个场景下，Allocatable 将会是 14.5 CPUs、28.5Gi 内存以及 88Gi 本地存储。 调度器保证这个节点上的所有 pod 请求的内存总量不超过 28.5Gi，存储不超过 88Gi。 当 pod 的内存使用总量超过 28.5Gi 或者磁盘使用总量超过 88Gi 时，Kubelet 将会驱逐它们。如果节点上的所有进程都尽可能多的使用 CPU，则 pod 加起来不能使用超过 14.5 CPUs 的资源。

当没有执行 kube-reserved 和/或 system-reserved 且系统守护进程使用量超过其预留时，如果节点内存用量高于 31.5Gi 或存储大于 90Gi，kubelet 将会驱逐 pod。