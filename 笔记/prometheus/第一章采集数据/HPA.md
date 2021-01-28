**Horizontal Pod Autoscaling**，简称HPA，是Kubernetes中实现POD水平自动伸缩的功能。它可以根据CPU使用率或应用自定义metrics自动扩展Pod数量(支持 replication controller、deployment 和 replica set)
### 
v1.11+HPA从metric-server获取监控数据
### **工作流程**
1.创建HPA资源，设定目标CPU使用率限额，以及最大、最小实例数, 一定要设置Pod的资源限制参数: request, 否则HPA不会工作。
2.控制管理器每隔30s(可以通过–horizontal-pod-autoscaler-sync-period修改)查询metrics的资源使用情况
3.然后与创建时设定的值和指标做对比(平均值之和/限额)，求出目标调整的实例个数
4.目标调整的实例数不能超过1中设定的最大、最小实例数，如果没有超过，则扩容；超过，则扩容至最大的实例个数