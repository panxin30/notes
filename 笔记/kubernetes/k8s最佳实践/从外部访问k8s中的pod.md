*   hostNetwork
*   hostPort
*   NodePort
*   LoadBalancer
*   Ingress

**说是暴露Pod其实跟暴露Service是一回事，因为Pod就是Service的backend。**

# hostNetwork: true

如果在Pod中使用`hostNetwork:true`配置的话，在这种pod中运行的应用程序可以直接看到pod启动的主机的网络接口。在主机的所有网络接口上都可以访问到该应用程序。
 
# hostPort

`hostPort`是直接将容器的端口与所调度的节点上的端口路由，这样用户就可以通过宿主机的IP加上来访问Pod了，如:。

这样做有个缺点，因为Pod重新调度的时候该Pod被调度到的宿主机可能会变动，这样就变化了，用户必须自己维护一个Pod与所在宿主机的对应关系。

这种网络方式可以用来做 nginx[Ingress controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx)。外部流量都需要通过kubenretes node节点的80和443端口。

# NodePort
要想让外部能够直接访问service，需要将service type修改为`nodePort`。
集群外就可以使用kubernetes任意一个节点的IP加上30000+端口访问该服务了。kube-proxy会自动将流量以round-robin的方式转发给该service的每一个pod。
