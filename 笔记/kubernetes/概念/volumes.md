```
SAN存储区域网络：iSCSI，
NAS网络xx存储：nfs，cifs
```
官网：https://kubernetes.io/zh/docs/concepts/storage/volumes/
https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/
**容器以 [subPath]卷挂载方式使用 ConfigMap 时，将无法接收 ConfigMap 的更新。**

kubectl explain pods.spec.env
```
配置容器化应用的方式：
        1、自定义命令行参数;
                args：[] (给出一些自定义参数)
        2、把配置文件直接打进镜像;
        3、环境变量
                （1）cloud native的应用程序一般可直接通过环境变量加载配置;
                （2）通过entrypoint脚本来预处理变量为配置文件中的配置信息
        4、存储卷
```
## **hostPath**
`hostPath`卷能将主机节点文件系统上的文件或目录挂载到您的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。

例如，`hostPath`的一些用法有：

*   运行一个需要访问 Docker 引擎内部机制的容器；请使用`hostPath`挂载`/var/lib/docker`路径。
*   在容器中运行 cAdvisor 时，以`hostPath`方式挂载`/sys`。
*   允许 Pod 指定给定的`hostPath`在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。

除了必需的`path`属性之外，用户可以选择性地为`hostPath`卷指定`type`。

支持的`type`值如下：
**DirectoryOrCreate**如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 Kubelet 相同的组和所有权。

**FileOrCreate**如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 Kubelet 相同的组和所有权。
## configmap-secret
两种特殊的存储卷，给我们管理员或用户提供了从集群外部向pod内部的应用注入配置信息
## 开发，QA，生产环境可能所使用的内存或cpu各不相同，因此得做3个镜像，这样太麻烦，因此可以用配置中心，起三份配置文件
## k8s也面临同样问题
因此我们不把配置文件写死在镜像中，启用configmap
1、启动多个pod的时候可以共享使用同一个configmap，用configmap关联到当前pod，从configmap中读取环境变量，每次启动pod时候获取
2、也可以把configmap当做存储卷，挂载在容器中的某个目录，这个目录恰好是应用程序读取配置信息的文件路径，支持动态修改。有时候可能需要手动重载
```
kubectl explain pods.spec.volumes.secret-同configmap作用差不多（不明文，使用加密）     
kubectl explain pods.spec.volumes.configMap-放的配置信息，也可以当存储卷（明文存储）
kubectl explain configMap
kubectl  create configmap --help 属于名称空间
```
pod-config.yaml
已经建立了一个configmap,名字叫nginx-config
`kubectl create configmap nginx-config --from-file=./www.conf` 文件名当键，文件内容当值
`kubectl create configmap citylist-config --from-file=/root/k8s-yaml/dump/clay/city-list/applicationContext.xml -n crm`
验证configmap
`kubectl get cm nginx-config -o yaml`
```
kubectl describe cm nginx-config -n ns
内容如下
nginx_port:
----
80
server_name:
----
myapp.magedu.com
```
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-1
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    magedu.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    env:
    - name: NGINX_SERVER_PORT
      valueFrom:
        configMapKeyRef:
          name: nginx-config
          key: nginx_port
    - name: NGINX_SERVER_NAME
      valueFrom:
        configMapKeyRef:
          name: nginx-config
          key: server_name
```
`kubectl exec -it pods -n default pod-cm-1` `printenv`打印环境变量验证configmap参数传递
`kubectl exec pods -n default pod-cm-1 -- printenv`
pod-config-2.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-2
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    magedu.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    volumeMounts:和ports一个级别，挂载哪些存储卷
    - name: nginxconf #挂谁， 要与 volume 中 name 匹配
      mountPath: /etc/nginx/config.d/挂哪里去，容器内路径
      readOnly: true不应该由我们的容器去修改它的内容
  volumes:
  - name: nginxconf存储卷名字
    configMap:存储卷类型
      name: nginx-config挂哪一个configmap
```
```
        volumeMounts:
        - mountPath: /proc/cpuinfo
          name: lxcfs-proc-cpuinfo
      volumes: 和containers一个级别
      - hostPath:
          path: /var/lib/lxcfs/proc/cpuinfo \# 使用 pod 所在节点的路径
          type: ""
        name: lxcfs-proc-cpuinfo
```
## 但是 pod 重启后， 如果漂移到其他节点， 那挂载的数据就会丢失，如果要求数据不能丢失，可以配合 nodeselector 使用。 即强制 pod 只运行在某个节点，重启或删除重建后数据不会丢失。
测试 kubectl exec -it pod-cm-2 -- /bin/sh
### 更新 ConfigMap 后：
*   使用该 ConfigMap 挂载的 Env**不会**同步更新
*   使用该 ConfigMap 挂载的 Volume 中的数据需要一段时间（实测大概10秒）才能同步更新
**2.secret**私钥或证书放在secret
kubectl create secret --help
```
Available Commands:
  docker-registry Create a secret for use with a Docker registry
    pod要拖取镜像，而镜像在需要认证在能拖取的私有镜像仓库，需要使用kubectl create secret  docker-registry 
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]
```
pod-secret-1.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-1
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    magedu.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-root-password
          key: password
```