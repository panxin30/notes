# 基础命令
```
create 通过文件名或标准输入创建资源
expose 将一个资源公开为一个新的k8s服务
run 创建并运行一个特定的镜像，可能是副本。创建一个deployment或job管理创建的容器
set 配置应用资源。修改现有应用程序资源
get 显式一个或多个资源。
explain 文档参考资料。
edit 使用默认的编辑器编辑一个资源
delete 通过文件名、标准输入、资源名称或标签选择器来删除资源
```
# 部署命令
```
rollout
rolling-update
scale
autoscale
```
# 直接命令扩充副本数
`kubectl scale deploy --replicas=0 tw-api wallex-api -n crm-prod`
# 直接以更新的方式重启pod 会新建pod代理老的pod
`kubectl -n kube-system rollout restart ds kube-proxy`
# 直接更新镜像
kubectl set image --help
`kubectl set image deploy bw-custom bw-custom=192.168.0.96:8082/bw-custom:v7.12.0 -n tbw-pre`
`kubectl get rs -n tbw-pre -o wide`
# 集群管理命令
```
certificate 修改证书资源
cluster-info 显式集群信息
top 显式资源(cpu/mermory/storage)使用。需要metrics-server运行
cordon 标记节点不可调度
uncordon 标记节点可调度
kubectl drain <node name>
在对节点执行维护（例如内核升级、硬件维护等）之前， 可以使用`kubectl drain`从节点安全地逐出所有 Pods。 安全的驱逐过程允许 Pod 的容器， 并确保满足指定的 PodDisruptionBudgets。
默认情况下，`kubectl drain`将忽略节点上不能杀死的特定系统 Pod
taint 增加污点
```
# 故障诊断和调试命令
```
describe 显式特定资源或资源组的详细信息
logs 在pod或指定的资源中容器打印日志
attach 附加到一个进程到一个已经运行的容器
exec 执行命令到容器
port-forward 转发一个或多个本地端口到一个pod
proxy 为k8s API Server启动代理服务
cp 拷贝文件或目录到容器中
auth 检查授权
```
# 高级命令
```
apply 通过文件名或标准输入对资源应用配置
patch 使用补丁修改、更新资源的字段
replace 通过文件名或标准输入替换一个资源
convert 不同的API版本之间转换配置文件。YAML和JSON格式都接受
```
# 设置命令
```
label 更新资源上的标签
kubectl label nodes <node-name> <label-key>=<label-value>  #打标签
kubectl get node --show-labels
annotate在一个或多个资源上更新注释
completion 用于实现kubectl工具自动补全
```
# 其他命令
```
api-versions 打印受支持的API版本
api-resources 打印所有已经注册的API资源
config 修改kubeconfig文件(用于访问API，比如配置认证信息)
help 所有命令帮助
plugin 运行一个命令行插件
version 打印客户端和服务版本信息
```

# 从容器中拷贝文件到宿主机
~~~
 kubectl cp  <some-namespace>/<some-pod>:/tmp/foo /tmp/bar
kubectl cp crm/inner-bw-pic-748cfd5c67-rxftl:/home/crm/inner-bw-pic.jar inner-bw-pic.jar
~~~

# 容器重启前的日志查看
`kubectl logs -p bw-facade-757649b6d6-2s4w7 -n crm-spre > 1.txt`
# 通过HostAliases修改pod的/etc/hosts文件
```
      dnsPolicy: ClusterFirst
      hostAliases:
      - hostnames:
        - vpn.51sw.cc
        ip: 192.168.60.179
```
# 列出所有命名空间下的所有容器

*   使用`kubectl get pods --all-namespaces`获取所有命名空间下的所有 Pod
*   使用`-o jsonpath={.items[*].spec.containers[*].image}`来格式化输出，以仅包含容器镜像名称。 这将以递归方式从返回的 json 中解析出`image`字段。
    *   参阅[jsonpath 说明](https://kubernetes.io/zh/docs/reference/kubectl/jsonpath/)获取更多关于如何使用 jsonpath 的信息。
*   使用标准化工具来格式化输出：`tr`,`sort`,`uniq`
    *   使用`tr`以用换行符替换空格
    *   使用`sort`来对结果进行排序
    *   使用`uniq`来聚合镜像计数

```
kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```
# 按 Pod 列出容器镜像，或者按名称空间列出镜像
可能能用于查看有么有pod镜像没更新到？
可以使用`range`操作进一步控制格式化，以单独操作每个元素。

```
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |\
sort
```
# 查看事件
`kubectl get events --all-namespaces`