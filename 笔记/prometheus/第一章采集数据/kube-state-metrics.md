官方文档：https://github.com/kubernetes/kube-state-metrics
### **概述**
metric-server，几乎容器运行的所有指标都能拿到，但是下面这种情况却无能为力：
我调度了多少个replicas？现在可用的有几个？
多少个Pod是running/stopped/terminated状态？
Pod重启了多少次？
我有多少job在运行中
而这些则是**kube-state-metrics**提供的内容，它基于client-go开发，轮询Kubernetes API，并将Kubernetes的结构化信息转换为metrics。
### 
使用kube-state-metrics后的常用场景有：
```
存在执行失败的Job: kube_job_status_failed{job="kubernetes-service-endpoints",k8s_app="kube-state-metrics"}==1
集群节点状态错误: kube_node_status_condition{condition="Ready",status!="true"}==1
集群中存在启动失败的Pod：kube_pod_status_phase{phase=~"Failed|Unknown"}==1
最近30分钟内有Pod容器重启: changes(kube_pod_container_status_restarts[30m])>0
```
配合报警可以更好地监控集群的运行

### **与metric-server的对比**
* metric-server是从api-server中获取cpu、内存使用率这种监控指标，并把他们发送给存储后端，如influxdb或云厂商，他当前的核心作用是：为HPA等组件提供决策指标支持。
* kube-state-metrics关注于获取k8s各种资源的最新状态，如deployment或者daemonset，之所以没有把kube-state-metrics纳入到metric-server的能力中，是因为他们的关注点本质上是不一样的。metric-server仅仅是获取、格式化现有数据，写入特定的存储，实质上是一个监控系统。而kube-state-metrics是将k8s的运行状况在内存中做了个快照，并且获取新的指标，但他没有能力导出这些指标
* 换个角度讲，kube-state-metrics本身是metric-server的一种数据来源，虽然现在没有这么做。
* 另外，像Prometheus这种监控系统，并不会去用metric-server中的数据，他都是自己做指标收集、集成的（Prometheus包含了metric-server的能力），但Prometheus可以监控metric-server本身组件的监控状态并适时报警，这里的监控就可以通过kube-state-metrics来实现，如metric-serverpod的运行状态。
