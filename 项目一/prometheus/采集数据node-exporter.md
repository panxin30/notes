# 在待监控的集群部署node-exporter和kube-state-metrics
```
   43  wget https://get.helm.sh/helm-v3.8.0-linux-amd64.tar.gz
   45  tar zxf helm-v3.8.0-linux-amd64.tar.gz 
         cd linux-amd64
   49  cp helm /usr/bin/
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install  kube-state-metrics prometheus-community/kube-state-metrics --namespace monitoring
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace monitoring
```
# 在promethues中配置监控指标node-exporter
可以通过kubernetes_sd_config从Kubernetes的REST API查找并拉取指标，并始终与集群状态保持同步。
kubernetes_sd_config的node和service都能发现node-exporter。
但这里使用file_sd_configs来发现node-exporter，因为node-exporter直接节点上暴露了端口9100
```
- job_name: k8s1-node-exporter
  file_sd_configs:
  - files:
    - '/etc/prometheus/file-sd-config/file-sd-k8s1.json'
```
# 直接制作文件夹为configmap
```
root@k8s2-01:/home/ubuntu/k8syaml/monitoring# cat file-sd-config/file-sd-k8s1.json 
 [                                                                                                       
  {
    "labels": {
      "hostname": "cn-office-tonytest-k8s1-1",
      "service": "k8s1"
    },
    "targets": [
      "192.168.60.204:9100"
    ]
  },
  {
    "labels": {
      "hostname": "cn-office-tonytest-k8s1-2",
      "service": "k8s1"
    },
    "targets": [
      "192.168.60.79:9100"
    ]
  }
]
```
`kubectl create configmap file-sd-config --from-file=file-sd-config/ -n monitoring` **将file-sd-config/目录下的文件都制作成configmap**
```
root@k8s2-01:/home/ubuntu/k8syaml/monitoring# kubectl get cm -n monitoring file-sd-config -o yaml
apiVersion: v1
data:
  file-sd-crm3qa.json: " [                                                                                                       \n
    \ {\n    \"labels\": {\n      \"hostname\": \"crm3qa-k8s-master\",\n      \"service\":
    \"crm3qa\"\n    },\n    \"targets\": [\n      \"192.168.60.81:9100\"\n    ]\n
    \ },\n  {\n    \"labels\": {\n      \"hostname\": \"crm3qa-k8s-slave01\",\n      \"service\":
    \"crm3qa\"\n    },\n    \"targets\": [\n      \"192.168.60.69:9100\"\n    ]\n
    \ },\n  {\n    \"labels\": {\n      \"hostname\": \"crm3qa-k8s-slave02\",\n      \"service\":
    \"crm3qa\"\n    },\n    \"targets\": [\n      \"192.168.60.153:9100\"\n    ]\n
    \ }\n]\n"
  file-sd-k8s1.json: " [                                                                                                       \n
    \ {\n    \"labels\": {\n      \"hostname\": \"cn-office-tonytest-k8s1-1\",\n      \"service\":
    \"k8s1\"\n    },\n    \"targets\": [\n      \"192.168.60.204:9100\"\n    ]\n  },\n
    \ {\n    \"labels\": {\n      \"hostname\": \"cn-office-tonytest-k8s1-2\",\n      \"service\":
    \"k8s1\"\n    },\n    \"targets\": [\n      \"192.168.60.79:9100\"\n    ]\n  }\n]\n"
```
# 将configmap挂载到prometheus-server指定目录下
```
        volumeMounts:
        - mountPath: /etc/prometheus/config
          name: config-volume
        - mountPath: /data
          name: storage-volume
        - mountPath: /etc/prometheus/file-sd-config
          name: file-sd-config
        - mountPath: /etc/prometheus/certs
          name: monitoring-certs
---
      volumes:
      - hostPath:
          path: /data/prometheus/
          type: Directory
        name: storage-volume
      - configMap:
          defaultMode: 420
          name: prometheus-server
        name: config-volume
      - configMap:
          defaultMode: 420
          name: file-sd-config
        name: file-sd-config
      - configMap:
          defaultMode: 420
          name: monitoring-certs
        name: monitoring-certs
```