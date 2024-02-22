 # cadvisor
现在要将`https://kubernetes.default.svc/api/v1/nodes/cn-office-tonytest-k8s-1/proxy/metrics/cadvisor`改造成为`https://192.168.60.204:6443/api/v1/nodes/cn-office-tonytest-k8s-1/proxy/metrics/cadvisor`
```
    - job_name: k8s1-cadvisor
      scrape_interval: 45s
      scrape_timeout: 20s
      metrics_path: /metrics
      kubernetes_sd_configs:
      - api_server: https://192.168.60.204:6443
        role: node
        tls_config:
          ca_file: /etc/prometheus/certs/ca.crt
          insecure_skip_verify: true
        bearer_token_file: /etc/prometheus/certs/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
        # 标签符合以__meta_kubernetes_node_label_开头的都留下
      - target_label: __address__
        replacement: 192.168.60.204:6443
      - target_label: __metrics_path__
        replacement: /api/v1/nodes/$1/proxy/metrics/cadvisor
        source_labels:
        - __meta_kubernetes_node_name
        regex: (.+)
      scheme: https
      tls_config:
        ca_file: /etc/prometheus/certs/ca.crt
        insecure_skip_verify: true
      bearer_token_file: /etc/prometheus/certs/token
```
# apiservers
```
    - job_name: k8s1-apiservers
      scrape_interval: 45s
      scrape_timeout: 20s
      metrics_path: /metrics
      kubernetes_sd_configs:
      - api_server: https://192.168.60.204:6443
        role: endpoints
        tls_config:
          ca_file: /etc/prometheus/certs/ca.crt
          insecure_skip_verify: true
        bearer_token_file: /etc/prometheus/certs/token
      relabel_configs:
      - action: keep
        regex: default;kubernetes;https
        source_labels:
        - __meta_kubernetes_namespace
        - __meta_kubernetes_service_name
        - __meta_kubernetes_endpoint_port_name
      scheme: https
      tls_config:
        ca_file: /etc/prometheus/certs/ca.crt
        insecure_skip_verify: true
      bearer_token_file: /etc/prometheus/certs/token
```