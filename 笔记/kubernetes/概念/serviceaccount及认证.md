官网： https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/
https://kubernetes.io/zh/docs/concepts/cluster-administration/certificates/
**在开启了 TLS 的集群中，每当与集群交互的时候少不了的是身份认证，使用 kubeconfig（即证书） 和 token 两种认证方式是最简单也最通用的认证方式**
Kubernetes: 认证、授权
```
API server:
    subject --> action --> object
认证:token,tls,user/password
    账号:UserAccount, ServiceAccount
授权:RBAC
    role, rolebinding
    clusterrole, clusterrolebinding
    rolebinding, clusterrolebinding:

    subject:
        user
        group
        serviceaccount
    role:
role, clusterrole
     object:
        resouce group
        resource
        non-resource url
    action:get, list, watch, patch, delete, deletecollection, ...
```
任何客户端经apiserver做操作之前得先完成认证-->授权检查-->准入控制()
RBAC----基于角色的访问控制机制，只有许可授权
客户端对apiserver发起请求
    user：username，uid
    group：
    extra：
    API
    Request path
        例子http://172.168.0.6:6443/apis/apps/v1/namespaces/default/deployments/myapp-deploy/对这个增删改查 
    HTTP request verb:
        get,post,put,delete
    API requets verb:
        get ,list,create,update,patch,watch,proxy,delete......
---    
k8s集群**有2类认证值**的用户账号
useraccount现实中的用户
serviceaccount---pod等应用程序想访问当前k8s集群的apiserver时里面要用到的认证信息。 
kubectl create serviceaccount mysa  新建个专用账号。
`kubectl create serviceaccount mysa -o yaml --dry-run` 干跑不执行，看看输出的格式
**导出POD配置信息**
`kubectl get pods -n crm bw-account-75f4c7db8f-hfl5l -o yaml --export`
**kubeconfig 是apiserver客户端连入apiserver时**
`kubectl confg view`
```
apiVersion: v1
clusters:
- cluster:集群列表
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.5.242:6443
  name: kubernetes
contexts:
- context:上下文列表
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes当前上下文
kind: Config
preferences: {}
users:
- name: kubernetes-admin用户列表
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```
**新建一个user**连接apiserver的帐号
1.私钥
`cd /etc/kubernetes/pki/`
`(umask 077; openssl genrsa -out tony.key 2048)`
2.生成csr签名请求文件
`openssl req -new -key tony.key -out tony.csr -subj "/CN=tony"`
3.用k8s的CA签证,使用 CA 证书及CA密钥 对请求签发证书进行签发，生成 x509证书
`openssl x509 -req -in tony.csr -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -out tony.crt -days 365`**默认都是一年**
4.验证证书
`openssl x509 -in tony.crt -text -noout`
5. 新建用户和上下文
`kubectl config set-credentials tony --client-certificate=./tony.crt --client-key=./tony.key --embed-certs=true`
`kubectl config set-context tony@kubernetes --cluster=kubernetes --user=tony`

7. 绑定权限
`kubectl create rolebinding crm --clusterrole=cluster-admin --user=tony`
8. 拷贝.kube/ 到普通用户tony
`cp -rp /root/.kube/ /home/tony/`,`chown -R tony.tony /home/tony/`
6. 切换用户
`kubectl config use-context tony@kubernetes`
9.验证只对crm名称空间有所有权限。
**授权插件**：Node，ABAC，RBAC，Webhook
    RBAC：Role-based AC 定义个角色，角色上绑定可以进行哪些操作
    角色（role）
    许可（）
role:
    operations
    objects
rolebinding:
    user account OR service account 
    role
**集群级别和名称空间级别**
clusterrole,clusterrolebinding 集群级别
Role,RoleBinding 名称空间级别
RoleBinding绑定clusterrole，也是名称空间级别的,这样就不用建多个role
`kubectl create role --help`
`kubectl create role pods-reader --verb=get,list,watch --resource=pods --dry-run -o yaml > pods-reader.yaml`
`kubectl create rolebinding --help`
`kubectl create rolebinding magedu-read-pods --role=pods-reader --user=magedu --dry-run - o yaml > readpods.yaml`
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    name: pods-reader
    namespace: default
rules:
  - apiGroups: 能对哪些api组内的操作
      - ""
    resources: 能对哪些资源进行操作
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:    能做什么操作
      - get
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups: 能对哪些api组内的操作
      - ""
    resources: 能对哪些资源进行操作
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs: 能做什么操作
      - list
      - watch 
 ---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef: 引用那个role
  apiGroup: rbac.authorization.k8s.io哪一类资源
  kind: Role
  name: nginx-ingress-role
subjects: 
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount授权给哪个用户
    namespace: ingress-nginx
```
kubectl describe rolebinding nginx-ingress-role-nisa-binding
*****
**拉取私有仓库镜像，认证的两种方式**
认证到pod或认证到serviceAccount
    在Deployment中，要拉取私有仓库镜像，就需要认证，建立一个secret，认证到pod中
imagePullSecrets:                                                                                                 
      - name: pull-secret
但是直接认证到pod的secret不太安全，可以考虑使用serviceAccountName(而serviceAccount可以附带认证到私有仓库的secret信息),定义imagePullSecrets到serviceaccount中，再把serviceAccount定义到pod中，这样我们的pod配置清单文件信息就不会泄漏出我们用的什么相关认证信息


 