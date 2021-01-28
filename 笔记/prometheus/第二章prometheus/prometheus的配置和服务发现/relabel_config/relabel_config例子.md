
然后是一段relabel_configs配置，其作用主要是用于匹配最终要抓取的endpoint，构造抓取的地址，甚至给最终的时序指标加上一些label。这里以apiserver发现的node_exporter endpoint信息为例，在没有relabel之前，其endpoint信息如下：
```
__address__="192.168.65.3:9100"
__meta_kubernetes_endpoint_address_target_kind="Pod"
__meta_kubernetes_endpoint_address_target_name="prometheus-node-exporter-89tcq"
__meta_kubernetes_endpoint_port_name="metrics"
__meta_kubernetes_endpoint_port_protocol="TCP"
__meta_kubernetes_endpoint_ready="true"
__meta_kubernetes_endpoints_name="prometheus-node-exporter"
__meta_kubernetes_namespace="default"
__meta_kubernetes_pod_container_name="prometheus-node-exporter"
__meta_kubernetes_pod_container_port_name="metrics"
__meta_kubernetes_pod_container_port_number="9100"
__meta_kubernetes_pod_container_port_protocol="TCP"
__meta_kubernetes_pod_controller_kind="DaemonSet"
__meta_kubernetes_pod_controller_name="prometheus-node-exporter"
__meta_kubernetes_pod_host_ip="192.168.65.3"
__meta_kubernetes_pod_ip="192.168.65.3"
__meta_kubernetes_pod_label_app="prometheus"
__meta_kubernetes_pod_label_component="node-exporter"
__meta_kubernetes_pod_label_controller_revision_hash="1203132889"
__meta_kubernetes_pod_label_pod_template_generation="1"
__meta_kubernetes_pod_label_release="prometheus"
__meta_kubernetes_pod_name="prometheus-node-exporter-89tcq"
__meta_kubernetes_pod_node_name="docker-for-desktop"
__meta_kubernetes_pod_ready="true"
__meta_kubernetes_pod_uid="62b53ba7-eae3-11e8-b0a6-025000000001"
__meta_kubernetes_service_annotation_prometheus_io_scrape="true"
__meta_kubernetes_service_label_app="prometheus"
__meta_kubernetes_service_label_chart="prometheus-7.4.1"
__meta_kubernetes_service_label_component="node-exporter"
__meta_kubernetes_service_label_heritage="Tiller"
__meta_kubernetes_service_label_release="prometheus"
__meta_kubernetes_service_name="prometheus-node-exporter"
__metrics_path__="/metrics"
__scheme__="http"
job="kubernetes-service-endpoints"
```
### 通过kubernetes_sd_configs发现机制，通过apiserver的接口列出当前k8s集群中endpoints列表
经过下面的relabel规则后：
```
    - job_name: kubernetes-service-endpoints
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - action: keep
        regex: true
        source_labels:
        - __meta_kubernetes_service_annotation_prometheus_io_scrape
# 只匹配__meta_kubernetes_service_annotation_prometheus_io_scrape=true的endpoint
# 然后进行下一步
# 如果有__meta_kubernetes_service_annotation_prometheus_io_scheme，且正则匹配(https?)，则将__scheme__修改为__meta_kubernetes_service_annotation_prometheus_io_scheme指定的值
      - action: replace
        regex: (https?)
        source_labels:
        - __meta_kubernetes_service_annotation_prometheus_io_scheme
        target_label: __scheme__
# 如果有__meta_kubernetes_service_annotation_prometheus_io_path，且正则匹配(.+)，则将__metrics_path__修改为__meta_kubernetes_service_annotation_prometheus_io_path指定的值
      - action: replace
        regex: (.+)
        source_labels:
        - __meta_kubernetes_service_annotation_prometheus_io_path
        target_label: __metrics_path__
 # 如果有__address__;__meta_kubernetes_service_annotation_prometheus_io_port，且正则匹配([^:]+)(?::\d+)?;(\d+)，则将__address__修改为$1:$2指定的值
      - action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        source_labels:
        - __address__
        - __meta_kubernetes_service_annotation_prometheus_io_port
        target_label: __address__
# 如果label名称正则匹配__meta_kubernetes_service_label_(.+)，则设置相应的label
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
 # 设置kubernetes_namespace为__meta_kubernetes_namespace的值
      - action: replace
        source_labels:
        - __meta_kubernetes_namespace
        target_label: kubernetes_namespace
# 设置kubernetes_name为__meta_kubernetes_service_name的值
      - action: replace
        source_labels:
        - __meta_kubernetes_service_name
        target_label: kubernetes_name
```
最终形成的抓取endpoint信息如下：
```
__address__="192.168.65.3:9100"
app="prometheus"
chart="prometheus-7.4.1"
component="node-exporter"
heritage="Tiller"
release="prometheus"
kubernetes_namespace="default"
kubernetes_service_name="prometheus-node-exporter"
__metrics_path__="/metrics"
__scheme__="http"
job="kubernetes-service-endpoints"
```
上述的endpoint信息交由prometheus，prometheus就可以得到抓取地址了，为`__scheme__://__address____metrics_path__`，也即`http://192.168.65.3:9100/metrics`