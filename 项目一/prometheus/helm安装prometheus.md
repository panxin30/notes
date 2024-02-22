# helm安装各种exporter
https://github.com/prometheus-community/helm-charts/tree/main/charts
Node-Exporter 不支持进程监控，可以加一个Process-Exporter
# helm 安装 prometheus
参考:https://artifacthub.io/packages/helm/prometheus-community/prometheus
参考:https://github.com/prometheus-community/helm-charts/tree/main/charts
前提：已有k8s环境
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo | grep prometheus
helm install prometheus prometheus-community/prometheus --namespace monitoring
helm uninstall prometheus  --namespace monitoring
# 安装版本为2.31.1
# 修改存储卷为本地存储
------------------------------------------------------------------------------------------------------------------------------------------------------

Prometheus 使用 kubernetes_sd_config 做服务发现，请求一般会经过集群的 Apiserver，随着规模的变大，**需要评估下对 Apiserver性能的影响，尤其是Proxy失败的时候**，会导致CPU 升高。
---
## **在监控Cadvisor、Docker、Kube-Proxy 的 Metric 时，我们一开始选择从 Apiserver Proxy 到节点的对应端口，统一设置比较方便，但后来还是改为了直接拉取节点，Apiserver 仅做服务发现。**

# 修改configmap的更新问题
kubectl edit cm -n monitoring prometheus-server
修改成功后，可能10来秒会自动更新
也可以使用reload立即更新
```
root@k8s2-01:/home/ubuntu# kubectl get svc -n monitoring
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
prometheus-server               ClusterIP   10.110.233.127   <none>        80/TCP     5d20h
root@k8s2-01:/home/ubuntu# curl -X POST "http://10.110.233.127:80/-/reload"
```
# 配置ingress
参考：https://kubernetes.io/zh/docs/concepts/services-networking/ingress/
```
root@k8s2-01:/home/ubuntu# kubectl get ingress -n monitoring prometheus-server -o yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
  name: prometheus-server
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: prometheus.51sw.cc
    http:
      paths:
      - backend:
          service:
            name: prometheus-server
            port:
              number: 80
        path: /
        pathType: Prefix
```