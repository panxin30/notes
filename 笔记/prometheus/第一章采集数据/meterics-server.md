
### **Metrics-Server**
Metrics server定时从Kubelet的Summary API(类似/ap1/v1/nodes/nodename/stats/summary)采集指标信息，这些聚合过的数据将存储在内存中，且以metric-api的形式暴露出去。
metric-server是从api-server中获取cpu、内存使用率这种监控指标kubectl top，并把他们发送给存储后端，如influxdb或云厂商，他当前的核心作用是：**为HPA等组件提供决策指标支持。**
**官方地址：**https://github.com/kubernetes-incubator/metrics-server
**部署metrics-server**
[https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/metrics-server](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/metrics-server)整合进了k8s中的，通过验证的
版本k8s,1.15.0,metrics0.3.3
修改下载后的清单
```
resource-reader.yaml

rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats    #新增这一行
  - namespaces
  verbs:
  - get
  - list
  - watch
####################################################
metrics-server-deployment.yaml

containers metrics-server 启动参数，修改好的如下：
 containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.3
        command:
        - /metrics-server
        - --metric-resolution=30s
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
        # These are needed for GKE, which doesn't support secure communication yet.
        # Remove these lines for non-GKE clusters, and when GKE supports token-based auth.
        #- --kubelet-port=10255
        #- --deprecated-kubelet-completely-insecure=true
----            
# 修改containers，metrics-server-nanny 启动参数，修改好的如下：
command:
          - /pod_nanny
          - --config-dir=/etc/config
          - --cpu=80m
          - --extra-cpu=0.5m
          - --memory=80Mi
          - --extra-memory=8Mi
          - --threshold=5
          - --deployment=metrics-server-v0.3.3
          - --container=metrics-server
          - --poll-period=300000
          - --estimator=exponential
          # Specifies the smallest cluster (defined in number of nodes)
          # resources will be scaled to.
          #- --minClusterSize={{ metrics_server_min_cluster_size }}
```

验证
~~~
kubectl get apiservices
~~~
`kubectl proxy --port=8080`
`curl http://localhost:8080/apis/metrics.k8s.io/v1veta1`
`curl http://localhost:8080/apis/metrics.k8s.io/v1veta1/nodes`
`curl http://localhost:8080/apis/metrics.k8s.io/v1veta1/pods`
kubectl top nodes
kubect top pods -n kube-system

