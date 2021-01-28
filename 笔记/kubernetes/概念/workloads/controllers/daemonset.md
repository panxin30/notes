## DaemonSet*确保全部（或者一些）Node 上运行一个 Pod 的副本。当有 Node 加入集群时，也会为他们新增一个 Pod 。当有 Node 从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

## 使用 DaemonSet 的一些典型用法：

*   运行集群存储 daemon，例如在每个 Node 上运行`glusterd`、`ceph`。
*   在每个 Node 上运行日志收集 daemon，例如`fluentd`、`logstash`。
*   在每个 Node 上运行监控 daemon，例如[Prometheus Node Exporter](https://github.com/prometheus/node_exporter)、`collectd`、Datadog 代理、New Relic 代理，或 Ganglia`gmond`。

一个简单的用法是，在所有的 Node 上都存在一个 DaemonSet，将被作为每种类型的 daemon 使用。 一个稍微复杂的用法可能是，对单独的每种类型的 daemon 使用多个 DaemonSet，但具有不同的标志，和/或对不同硬件类型具有不同的内存、CPU要求。
ds-demo.yaml
kubectl expose deployment redis --port=6379
```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: redis
    namespace: default
spec:
    replicas: 1
    selector:
        matchLabels:
            app: redis
            role: logstor
template:
    metadata:
        labels:
            app: redis
            role: logstor
    spec:（pod的）
        containers:
        - name: redis
          image: redis:4.0-alpline
          ports:
          - name: redis
            containerPort: 6379
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat-ds
  namespace: default
spec:
  selector:
    matchLabels:
      app: filebeat
      release: stable
  template:
    metadata:
      labels:
        app: filebeat
        release: stable
    spec:
      containers:
      -name: filebeat
      image: ikubernetes/filebeat:5.6.5-alpine
      env:
      - name: REDIS_HOST
        value: redis.default.svc.cluster.local
      - name: REDIS_LOG_LEVEL
        value: info
```