# 使用kubernetes_sd_configs，不管service、service endpoint、pod、node最终目的是要获取到metrics

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

# 在prometheus中配置监控目标kube-state-metrics
在安装kube-state-metrics的时候自动创建了用户
```
root@cn-office-tonytest-k8s-1:/home/ubuntu# kubectl get sa -n monitoring
NAME                                     SECRETS   AGE
kube-state-metrics                       1         3d15h
root@cn-office-tonytest-k8s-1:/home/ubuntu# kubectl get ClusterRole/kube-state-metrics
NAME                 CREATED AT
kube-state-metrics   2022-03-10T09:39:41Z
```
在这个用户的权限上增加1个权限用于kube-apiserver以及后面发现还需要添加一个操作对象和操作动作
```
kubectl edit ClusterRole/kube-state-metrics
- nonResourceURLs: 
  - /metrics
  verbs:
  - get
# 允许在非资源端点"/metrics"发起 GET请求（必须在 ClusterRole 绑定 ClusterRoleBinding 才生效）
```
# apiserver授权
要访问K8S apiserver需要先进行授权，集群内部Prometheus可以使用集群内默认配置进行访问。而集群外访问需要使用token+客户端cert进行认证，因此需要先进行RBAC授权。
**取待监控的k8s集群的CA证书**(/etc/kubernetes/pki/ca.crt)为prometheus配置文件中该job的CA，**取待监控的k8s用户**kube-state-metrics的secret中的token 为prometheus配置文件中该job的token
```
root@cn-office-tonytest-k8s-1:/home/ubuntu# kubectl get sa -n monitoring kube-state-metrics -o yaml
secrets:
- name: kube-state-metrics-token-cp96g
root@cn-office-tonytest-k8s-1:/home/ubuntu# kubectl describe secret -n monitoring kube-state-metrics-token-cp96g
Data
====
ca.crt:     1099 bytes
namespace:  10 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Im4tR0N0WS1nLUk3U0VYbjhWT1B6bjBjNmpUVF9IYUtNWlFWSGNhZFljaUUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJtb25pdG9yaW5nIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Imt1YmUtc3RhdGUtbWV0cmljcy10b2tlbi1jcDk2ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlLXN0YXRlLW1ldHJpY3MiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJjYTZjZWUyMi1iMTZlLTQ5NWEtODEzNy1jMzgzMWM4OWQ2MjUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6bW9uaXRvcmluZzprdWJlLXN0YXRlLW1ldHJpY3MifQ.uOrDxlOTnhES3e8fFUXd57XklVArqON9u108yI9kHCt6fCZZJUTncFo4tyH6iuMW2nx7ASfUHWj_WhZs_F3nl8bGh7jtSGIJmVOPVt9mOJ3iMXS2YkfvYQK_hpxujHduEw6L7bF77fHXh9qmpLUdAgpvtIgfbAkJSk62JL2Jpikwo94BWE5pTkhBxdRTH-9HN14jD4CsxzuxhaLumdBmcUpJCeTclg2KDmJZgymCgC-8OEq08nGnMr0JtUHF_J8PDBaIh7BpordI4z9rNwSOAFLMFp0sSOFKzW3OJHSB4uCD-UOBSX_KVA0nEnddEHRvQN4vLJcWbh3_WBqBkRjiyw
```
# 待监控机器的token和ca复制到prometheus所在集群中
创建对应名字文件ca.crt token
kubectl create configmap test --from-file=ca.crt --from-file=token -n monitoring
kubectl get cm -n monitoring test -o yaml
将2条数据复制到正在使用的monitoring-certs中
kubectl edit cm -n monitoring monitoring-certs中，因为手动修改这个cm容易出错
### **另外 kubectl get secret出来的是base64加密后的，kubectl describe secret 获取的是不加密的。**

# 集群外部署Prometheus和集群内部署Prometheus是不一样的
**集群外Prometheus使用默认自动拼接的监控url是无法访问的**，**此时需要自行构造apiserver proxy URLs**，通过proxy url,集群外Prometheus就可以访问监控url来拉取监控指标了。
# 先来看看没有自行构造前是什么样子的
就是未启用relabel_configs
```
kubectl edit cm -n monitoring prometheus-server
    - job_name: k8s1-kube-state-metrics
      scrape_interval: 45s
      scrape_timeout: 20s
      metrics_path: /metrics
      kubernetes_sd_configs:
      - api_server: https://192.168.60.204:6443
        role: service
        tls_config:
          ca_file: /etc/prometheus/certs/ca.crt
          insecure_skip_verify: true
        bearer_token_file: /etc/prometheus/certs/token
      tls_config:
        ca_file: /etc/prometheus/certs/ca.crt
        insecure_skip_verify: true
      bearer_token_file: /etc/prometheus/certs/token
```
# **未启用relabel_configs状态见附录**
http://prometheus.51sw.cc:30937/ 点击status-->选择targets-->k8s1-kube-state-metrics可以看到是默认的监控URL，状态为DOWN。
URL为 `http://kube-state-metrics.monitoring.svc:8080/metrics`
现在是要将`http://kube-state-metrics.monitoring.svc:8080/metrics`改造成为`https://192.168.60.204:6443/api/v1/namespaces/monitoring/services/http:kube-state-metrics:8080/proxy/metrics`
为什么改造成这样，prometheus的默认有监控pod，node，service的配置，监控页面的target能看见

```
kubectl edit cm -n monitoring prometheus-server
    - job_name: k8s1-kube-state-metrics
      scrape_interval: 45s
      scrape_timeout: 20s
      metrics_path: /metrics
      kubernetes_sd_configs:
      - api_server: https://192.168.60.204:6443
        role: service
        tls_config:
          ca_file: /etc/prometheus/certs/ca.crt
          insecure_skip_verify: true
        bearer_token_file: /etc/prometheus/certs/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_name]
        action: keep
        regex: '^(kube-state-metrics)$'
        # 只保留指定匹配正则的标签，不匹配则删除
      - target_label: __address__
        replacement: 192.168.60.204:6443
        # 使用replacement值替换__address__默认值
      - target_label: __metrics_path__
        replacement: /api/v1/namespaces/monitoring/services/http:${1}:8080/proxy/metrics
        # 使用replacement值替换__metrics_path__默认值
        source_labels:
        - __meta_kubernetes_service_name
        regex: (.+)
        # 从哪里取${1}这个值
      scheme: https
      # scheme改为https
      tls_config:
        ca_file: /etc/prometheus/certs/ca.crt
        insecure_skip_verify: true
      bearer_token_file: /etc/prometheus/certs/token
```

# 测试
```
root@k8s2-01:/home/ubuntu/k8syaml/monitoring# curl -s https://192.168.60.204:6443/api/v1/namespaces/monitoring/services/http:kube-state-metrics:8080/proxy/metrics --header "Authorization: Bearer $token" --cacert certs/ca.crt 
#token就是上面的token，ca就是上面的ca
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "services \"http:kube-state-metrics:8080\" is forbidden: User \"system:serviceaccount:monitoring:kube-state-metrics\" cannot get resource \"services/proxy\" in API group \"\" in the namespace \"monitoring\"",
  "reason": "Forbidden",
  "details": {
    "name": "http:kube-state-metrics:8080",
    "kind": "services"
  },
  "code": 403
}
```
根据提示增加一个可操作资源services/proxy，动作为get
```
kubectl edit ClusterRole/kube-state-metrics
- apiGroups:
  - ""
  resources:
  - services/proxy
  - nodes/proxy
  - pods/proxy
  verbs:
  - get
  - list
  - watch
```
这下成功了。
# 测试下能不能获取到node，pod的状态
在graph录入下面的代码
kube_pod_status_phase
kube_node_status_condition

# 附录

```
__address__="kube-state-metrics.monitoring.svc:8080"
__meta_kubernetes_namespace="monitoring"
__meta_kubernetes_service_annotation_meta_helm_sh_release_name="kube-state-metrics"
__meta_kubernetes_service_annotation_meta_helm_sh_release_namespace="monitoring"
__meta_kubernetes_service_annotation_prometheus_io_scrape="true"
__meta_kubernetes_service_annotationpresent_meta_helm_sh_release_name="true"
__meta_kubernetes_service_annotationpresent_meta_helm_sh_release_namespace="true"
__meta_kubernetes_service_annotationpresent_prometheus_io_scrape="true"
__meta_kubernetes_service_cluster_ip="10.96.98.2"
__meta_kubernetes_service_label_app_kubernetes_io_component="metrics"
__meta_kubernetes_service_label_app_kubernetes_io_instance="kube-state-metrics"
__meta_kubernetes_service_label_app_kubernetes_io_managed_by="Helm"
__meta_kubernetes_service_label_app_kubernetes_io_name="kube-state-metrics"
__meta_kubernetes_service_label_app_kubernetes_io_part_of="kube-state-metrics"
__meta_kubernetes_service_label_app_kubernetes_io_version="2.4.1"
__meta_kubernetes_service_label_helm_sh_chart="kube-state-metrics-4.7.0"
__meta_kubernetes_service_labelpresent_app_kubernetes_io_component="true"
__meta_kubernetes_service_labelpresent_app_kubernetes_io_instance="true"
__meta_kubernetes_service_labelpresent_app_kubernetes_io_managed_by="true"
__meta_kubernetes_service_labelpresent_app_kubernetes_io_name="true"
__meta_kubernetes_service_labelpresent_app_kubernetes_io_part_of="true"
__meta_kubernetes_service_labelpresent_app_kubernetes_io_version="true"
__meta_kubernetes_service_labelpresent_helm_sh_chart="true"
__meta_kubernetes_service_name="kube-state-metrics"
__meta_kubernetes_service_port_name="http"
__meta_kubernetes_service_port_protocol="TCP"
__meta_kubernetes_service_type="ClusterIP"
__metrics_path__="/metrics"
__scheme__="https"
__scrape_interval__="45s"
__scrape_timeout__="20s"
job="k8s1-kube-state-metrics"
```