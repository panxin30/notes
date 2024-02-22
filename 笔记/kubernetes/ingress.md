# Ingress组成

Ingress由三部分组成：

*   反向代理负载均衡器
    
    比如Nginx、Haproxy、Apache、traefik等
    
*   Ingress Controller
    
    通过与 Kubernetes API 交互，动态的去感知集群中 Ingress 规则变化，然后读取它，按照自定义的规则，规则就是写明了哪个域名对应哪个service，生成一段 Nginx 配置，再写到 Nginx-ingress-control的 Pod 里，这个 Ingress Contronler 的pod里面运行着一个nginx服务，控制器会把生成的nginx配置写入/etc/nginx.conf文件中，然后 reload 一下 使用配置生效。以此来达到域名分配置及动态更新的问题。
    
*   Ingress
    
    kubernetes的一个资源对象，用于编写定义规则

# 安装指南
https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md

## 一、Ingress Controller 使用 Deployment 部署

Using[NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport):

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml
```
直接下载了deploy.yaml，修改service为nodeport
```
spec:
  type: NodePort
  ports:
    - name: http
      nodePort: 30080
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      nodePort: 30443
      port: 443
      protocol: TCP
      targetPort: https
```
## 二、Ingress Controller 使用 DeamonSet 部署，Pod 指定 hostPort 来暴露端口undefined优点：免费undefined缺点：没有高可用保证，如果需要高可用就得自己去搞
### **代理的service必需是无头服务？？？**
![](../images/screenshot_1565764264707.png)
 #证书需要转化为secret
`kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls,key`
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
    name: ingress-tomcat-tls
    namespace: default
    annotations:注解，k8s才能是别到对应的规则
        kubernetes.io/ingress.class: "nginx" 才能转化为相匹配的controller规则
spec:
    tls:
    - hosts:
      - tomcat.magedu.com
      secretName: tomcat-ingress-secret
    rules:定义规则
    - host: tomcat.magedu.com
      http:
        paths:
        - path: / 
          backend:
            serviceName: tomcat
            servicePort: 8080
```
**ingress在安装时候指定了nodeport为30080和30443，所以访问时候应该域名加端口
跳板机nginx所有80和443指向ingress所在服务器内网的30080和30443**
# 三、k8s1.19后ingress配置有变动
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gocd-server
  namespace: gocd
spec:
  ingressClassName: nginx
  rules:
    - host: gocd.51sw.cc
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gocd-server
                port:
                  number: 8153
```