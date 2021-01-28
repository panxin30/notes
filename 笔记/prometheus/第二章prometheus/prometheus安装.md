### 几种部署方式，从手动部署到更自动化部署
Single binary -> Docker container -> Kubernetes Deployment -> the Helm Kubernetes package manager->Prometheus operator
### Note that you can easily adapt this Docker container into a proper Kubernetes Deployment object that will mount the configuration from a ConfigMap, expose a service, deploy multiple replicas, etc.
### **helm安装prometheus和grafana**
前提已经安装了helm3.x版本
```
helm search repo stable | grep grafana
helm search repo stable | grep prometheus
helm search repo stable | grep nginx-ingress
helm pull stable/prometheus
helm pull stable/grafana
helm install  stable/prometheus --generate-name
helm install stable/grafana --generate-name
```
### **k8s Deployment的方式部署**
**部署prometheus**
版本k8s,1.17.0,alertmanager0.20.0  kube-state-metrics1.9.5  node_exporter0.18.1  prometheus2.16.0
For production use check out more mature setups like[Prometheus Operator](https://github.com/coreos/prometheus-operator)and[kube-prometheus](https://github.com/coreos/kube-prometheus).推荐使用
*****
组成部分**alertmanager  kube-state-metrics  node_exporter  prometheus 。 pushgateway可选**

4.安装k8s-prometheus-adapter，需要用到https，需要k8s认可的CA签署的证书.安装这个是为了HPA自动伸缩。光监控应该不用安装
        1.私钥
`cd /etc/kubernetes/pki/`
`(umask 077; openssl genrsa -out serving.key 2048)`
        2.生成csr签名请求文件
`openssl req -new -key serving.key -out serving.csr -subj "/CN=serving"`
`openssl req -new -key serving.key -out serving.csr -subj "/C=CN/ST=GD/L=SZ/O=vihoo/OU=dev/CN=vivo.com/emailAddress=yy@vivo.com"`
        3.用CA签证,使用 CA 证书及CA密钥 对请求签发证书进行签发，生成 x509证书
`openssl x509 -req -in serving.csr -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -out serving.crt -days 3650`**默认都是一年**
        4. 创建secret
        `kubectl create secret generic cm-adapter-serving-certs --from-file=./serving.crt --from-file=./serving.key -n monitoring`
        5.验证
        `kubectl get all -n monitoring`
5.安装grafana，可选 
grafana官网 https://grafana.com/docs/grafana/latest/auth/overview/#anonymous-authentication
[https://github.com/coreos/kube-prometheus/blob/master/manifests/grafana-deployment.yaml](https://github.com/coreos/kube-prometheus/blob/master/manifests/grafana-deployment.yaml)
部分修改
`kubectl create secret tls grafana.test.com --cert=./grafana.crt --key=./test.key -n monitoring`
`kubectl create secret tls prometheus.test.com --cert=./prometheus.crt --key=./test.key -n monitoring`

```
    spec:
      containers:
      - env:
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "no"
        - name: GF_SECURITY_ADMIN_PASSWORD 
          value: "highlowtop"

        volumeMounts:
        - mountPath: /var/lib/grafana
          name: grafana-storage
          subPath: grafana-storage
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      volumes:
      - hostPath:
          path: /etc/ssl/certs
          type: ""
        name: ca-certificates
      - hostPath:
          path: /data/
          type: Directory
        name: grafana-storage
```
grafana 选择datasource的时候填写的prometheus在集群中的svc地址
`http://prometheus-server.monitoring.svc.cluster.local:80`
6 ingress
nginx-443->ingress->grafana
根据上面的证书添加到nginx
