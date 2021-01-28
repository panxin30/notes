**官网**：https://kubernetes.io/zh/docs/concepts/services-networking/service/
**服务发现与负载均衡**：https://developer.aliyun.com/article/728115?utm_content=g_1000089781
**CNI**
    + flannel:网络配置
    + calico:网络配置，网络策略
    + canel :前两种搭配使用
### **创建service的动机**
Kubernetes PodsPod 是 Kubernetes 的原子对象。Pod 表示您的集群上一组正在运行的容器。 是有生命周期的。他们可以被创建，而且销毁不会再启动。 如果您使用 Deployment 来运行您的应用程序，则它可以动态创建和销毁 Pod。

每个 Pod 都有自己的 IP 地址，但是在 Deployment 中，在同一时刻运行的 Pod 集合可能与稍后运行该应用程序的 Pod 集合不同。

这导致了一个问题： 如果一组 Pod（称为“后端”）为群集内的其他 Pod（称为“前端”）提供功能，那么前端如何找出并跟踪要连接的 IP 地址，以便前端可以使用工作量的后端部分？
### **service**
Kubernetes Service 定义了外界访问一组特定 Pod 的方式，这一组 Pod 能够被 Service 访问到，通常是通过 selector 实现的。Service 有自己的 IP 和端口，并把这个IP和后端的Pod所跑的服务的关联起来。Service 为 Pod 提供了负载均衡。
    工作模式：userspace,iptables,ipvs
        ipvs:1.11+
    类型：
        ExternalName,ClusterIP,NodePort,LoadBalancer
### **定义service**
myapp-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
    name: myapp
    namespace: default
spec:
    selector:
        app: myapp
        release: canary
    clusterIP: 10.98.98.98可以不指定，会自动分配，指定的情况下，要保证不冲突
    type: NodePort
    ports:
    - port: 80service监听的端口
      targetPort: 80容器的端口
      nodePort: 30080保证不被其他service占用，service ip上的端口映射为nodePort30080。节点的端口
```
### **过程**
`NodePort: client->NodeIP:NodePORT->ClusterIP:ServicePort->PodIP:containerPort`
### **没有 selector 的 Service**

*   希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。
*   希望服务指向另一个[命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces)中或其它集群中的服务。
*   您正在将工作负载迁移到 Kubernetes。 在评估该方法时，您仅在 Kubernetes 中运行一部分后端。
在任何这些场景中，都能够定义没有 selector 的`Service`。 实例:
```
apiVersion: v1
kind: Service
metadata:
  name: saas-work-dev
  namespace: saas-work-dev
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: ClusterIP
```
### 由于**此服务没有选择器**，因此**不会**自动创建相应的 **Endpoint** 对象。 您可以通过**手动添加 Endpoint** 对象，将服务手动映射到运行该服务的网络地址和端口
```
apiVersion: v1
kind: Endpoints
metadata:
  name: saas-work-dev
  namespace: saas-work-dev
subsets:
  - addresses:
      - ip: 192.168.60.227
    ports:
      - port: 80
        protocol: TCP
```
访问没有 selector 的`Service`，与有 selector 的`Service`的原理相同。 请求将被路由到用户定义的 Endpoint， YAML中为:192.168.60.227:80
### **Headless Services**
引用：https://zhuanlan.zhihu.com/p/54153164
*   第一种：自主选择权，有时候`client`想自己来决定使用哪个`Real Server`，可以通过查询`DNS`来获取`Real Server`的信息。
*   第二种：`Headless Services`还有一个用处（PS：也就是我们需要的那个特性）。`Headless Service`的对应的每一个`Endpoints`，即每一个`Pod`，都会有对应的`DNS`域名；这样`Pod`之间就可以互相访问。
`web`为我们创建的`StatefulSets`，对应的`pod`的域名为`web-0`，`web-1`，他们之间可以互相访问，这样对于一些集群类型的应用就可以解决互相之间身份识别的问题了。

### **ExternalName**
类型为 ExternalName 的服务将服务映射到 DNS 名称，而不是典型的选择器，例如`my-service`,您可以使用`spec.externalName`参数指定这些服务。

例如，以下 Service 定义将`prod`名称空间中的`my-service`服务映射到`my.database.example.com`：

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
~~~
### **ServiceTypes**
Kubernetes`ServiceTypes`允许指定一个需要的类型的 Service，默认是`ClusterIP`类型。

`Type`的取值以及行为如下：

*   `ClusterIP`：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的`ServiceType`。
*   [`NodePort`](https://kubernetes.io/zh/docs/concepts/services-networking/service/#nodeport)：通过每个 Node 上的 IP 和静态端口（`NodePort`）暴露服务。`NodePort`服务会路由到`ClusterIP`服务，这个`ClusterIP`服务会自动创建。通过请求`<NodeIP>:<NodePort>`，可以从集群的外部访问一个`NodePort`服务。
*   [`LoadBalancer`](https://kubernetes.io/zh/docs/concepts/services-networking/service/#loadbalancer)：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到`NodePort`服务和`ClusterIP`服务。
*   [`ExternalName`](https://kubernetes.io/zh/docs/concepts/services-networking/service/#externalname)：通过返回`CNAME`和它的值，可以将服务映射到`externalName`字段的内容（例如，`foo.bar.example.com`）。 没有任何类型代理被创建。
    
    > 注意：您需要 CoreDNS 1.7 或更高版本才能使用`ExternalName`类型。

您也可以使用[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)来暴露自己的服务。 Ingress 不是服务类型，但它充当集群的入口点。 它可以将路由规则整合到一个资源中，**因为它可以在同一IP地址下公开多个服务。**





需求，k8s集群中一个pod client要访问集群外部的一个服务
通过service代理到外部服务。外部服务先转给nodeIP再转给service，再转给内部pod client


    资源记录：
        SVC_NAME.NS_NAME.DOMAIN.LTD.
        redis.default.svc.cluster.local.
service到endpoint到pod
`kubectl explain svc.spec.ports`
    - port: 80service监听的端口
      targetPort: 80容器的端口
      nodePort: 30080保证不被其他service占用，service ip上的端口映射为nodePort30080。节点的端口
redis-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
    name: redis
    namespace: default
spec:
    selector:
        app: redis
        role: logstor
    clusterIP: 10.97.97.97可以不指定，会自动分配，指定的情况下，要保证不冲突
    type: ClusterIP(是私网地址，因此只集群内部可达，可以被各pod访问，被节点本身所访问)
    ports:
    - port: 6379 service端口
      targetPort: 6379容器端口
```
