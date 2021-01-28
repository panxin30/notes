### **概述**
Prometheus可以从Kubernetes集群的各个组件中采集数据，比如kubelet中自带的cadvisor，api-server等，而node-export就是其中一种来源
### 
Exporter是Prometheus的一类数据采集组件的总称。它负责从目标处搜集数据，并将其转化为Prometheus支持的格式。与传统的数据采集组件不同的是，它并不向中央服务器发送数据，而是等待中央服务器主动前来抓取，默认的抓取地址为http://CURRENT_IP:9100/metrics
### 
node-exporter用于采集服务器层面的运行指标，包括机器的loadavg、filesystem、meminfo（cpu 负载，内存使用，磁盘 IO，网络）等基础监控，类似于传统主机监控维度的zabbix-agent
node-export由prometheus官方提供、维护，不会捆绑安装，但基本上是必备的exporter
### **部署**
node-exporter.yaml文件：
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  labels:
    app: node-exporter
    name: node-exporter
  name: node-exporter
spec:
  clusterIP: None
  ports:
  - name: scrape
    port: 9100
    protocol: TCP
  selector:
    app: node-exporter
  type: ClusterIP
----
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  template:
    metadata:
      labels:
        app: node-exporter
      name: node-exporter
    spec:
      containers:
      - image: registry.cn-hangzhou.aliyuncs.com/tryk8s/node-exporter:latest
        name: node-exporter
        ports:
        - containerPort: 9100
          hostPort: 9100
          name: scrape
      hostNetwork: true
      hostPID: true
```