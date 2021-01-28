将Pod分配给指定的节点 - Kubernetes
官网 https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
你可以将Pod限制到某个特定节点上运行，或者更倾向在特定节点上运行。有几种方法可以做到这一点，推荐的方法都使用标签选择器来进行选择。

大多数情况，约束是不必要的，因为调度程序会自动进行合理的放置（例如，将pod分布在资源充足的节点上，而不是将节点放在没有足够资源的节点上）。

但是在某些情况下，你可能需要做特别的控制，例如，将pod放置有 SSD 的节点上，或者将两个不同服务中的pod共同定位到同一个区域中。

nodeSelector定向调度
内置的节点标签 
~~~yaml
  nodeName: kube-01
~~~
使用`nodeName`来选择节点的一些限制：

*   如果指定的节点不存在，
*   如果指定的节点没有资源来容纳 pod，pod 将会调度失败并且其原因将显示为，比如 OutOfmemory 或 OutOfcpu。
*   云环境中的节点名称并非总是可预测或稳定的。
#
节点隔离/限制
亲和和反亲和
nodeName调度
污点和容忍