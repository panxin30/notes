### **服务发现**
### `kubernetes_sd_config` 这个是以角色(role)来定义收集的，Kubernetes SD配置允许从[Kubernetes](https://kubernetes.io/)的RESTAPI中检索scrape目标，并始终与群集状态保持同步。
凡`<role>`必须是`endpoints`，`service`，`pod`，`node`，或`ingress`。
`role`可以配置以下类型之一来发现目标：
## **node**

1.  `node` `node`角色发现每个群集节点有一个目标，其地址默认为Kubelet的HTTP端口。 目标地址默认为`NodeInternalIP`，`NodeExternalIP`，`NodeLegacyHostIP`和`NodeHostName`的**地址类型顺序中Kubernetes节点对象的第一个现有地址**。

可用元标签：

*   `__meta_kubernetes_node_name`：节点对象的名称。
*   `__meta_kubernetes_node_label_<labelname>`：节点对象中的每个标签。
*   `__meta_kubernetes_node_labelpresent_<labelname>`：`true`对于节点对象中的每个标签。
*   `__meta_kubernetes_node_annotation_<annotationname>`：来自节点对象的每个注释。
*   `__meta_kubernetes_node_annotationpresent_<annotationname>`：`true`对于节点对象中的每个注释。
*   `__meta_kubernetes_node_address_<address_type>`：每个节点地址类型的第一个地址（如果存在）。

此外，`instance`节点的标签将设置为从API服务器检索的节点名称。
##  **service**

该`service`角色为每个服务发现每个服务端口的目标。这对于服务的黑盒监控通常很有用。该地址将设置为服务的Kubernetes DNS名称和相应的服务端口。

可用元标签：

*   `__meta_kubernetes_namespace`：服务对象的命名空间。
*   `__meta_kubernetes_service_annotation_<annotationname>`：来自服务对象的每个注释。
*   `__meta_kubernetes_service_annotationpresent_<annotationname>`：“true”表示服务对象的每个注释。
*   `__meta_kubernetes_service_cluster_ip`：服务的群集IP地址。（不适用于ExternalName类型的服务）
*   `__meta_kubernetes_service_external_name`：服务的DNS名称。（适用于ExternalName类型的服务）
*   `__meta_kubernetes_service_label_<labelname>`：来自服务对象的每个标签。
*   `__meta_kubernetes_service_labelpresent_<labelname>`：`true`对于服务对象的每个标签。
*   `__meta_kubernetes_service_name`：服务对象的名称。
*   `__meta_kubernetes_service_port_name`：目标服务端口的名称。
*   `__meta_kubernetes_service_port_protocol`：目标服务端口的协议。
##  **pod**

该`pod`角色发现所有pod并将其容器公开为目标。对于容器的每个声明端口，将生成单个目标。如果容器没有指定端口，则会创建每个容器的无端口目标，以通过重新标记手动添加端口。

可用元标签：

*   `__meta_kubernetes_namespace`：pod对象的命名空间。
*   `__meta_kubernetes_pod_name`：pod对象的名称。
*   `__meta_kubernetes_pod_ip`：pod对象的pod IP。
*   `__meta_kubernetes_pod_label_<labelname>`：pod对象中的每个标签。
*   `__meta_kubernetes_pod_labelpresent_<labelname>`：`true`对于pod对象中的每个标签。
*   `__meta_kubernetes_pod_annotation_<annotationname>`：pod对象中的每个注释。
*   `__meta_kubernetes_pod_annotationpresent_<annotationname>`：`true`对于pod对象中的每个注释。
*   `__meta_kubernetes_pod_container_init`：`true`如果容器是[InitContainer](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
*   `__meta_kubernetes_pod_container_name`：目标地址指向的容器的名称。
*   `__meta_kubernetes_pod_container_port_name`：容器端口的名称。
*   `__meta_kubernetes_pod_container_port_number`：容器端口号。
*   `__meta_kubernetes_pod_container_port_protocol`：容器端口的协议。
*   `__meta_kubernetes_pod_ready`：设置为`true`或`false`准备好pod的就绪状态。
*   `__meta_kubernetes_pod_phase`：设置为`Pending`，`Running`，`Succeeded`，`Failed`或`Unknown`在[生命周期](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)。
*   `__meta_kubernetes_pod_node_name`：pod安排到的节点的名称。
*   `__meta_kubernetes_pod_host_ip`：pod对象的当前主机IP。
*   `__meta_kubernetes_pod_uid`：pod对象的UID。
*   `__meta_kubernetes_pod_controller_kind`：对象类型的pod控制器。
*   `__meta_kubernetes_pod_controller_name`：pod控制器的名称。
## **endpoints**

该`endpoints`角色从列出的服务端点发现目标。对于每个端点地址，每个端口发现一个目标。如果端点由pod支持，则pod的所有其他容器端口（未绑定到端点端口）也会被发现为目标。

可用元标签：

*   `__meta_kubernetes_namespace`：端点对象的命名空间。
*   `__meta_kubernetes_endpoints_name`：端点对象的名称。对于直接从端点列表中发现的所有目标（不是从底层pod中另外推断的那些），附加以下标签：
*   `__meta_kubernetes_endpoint_hostname`：端点的主机名。
   *   `__meta_kubernetes_endpoint_node_name`：托管端点的节点的名称。
   *   `__meta_kubernetes_endpoint_ready`：设置为`true`或`false`为端点的就绪状态。
   *   `__meta_kubernetes_endpoint_port_name`：端点端口的名称。
   *   `__meta_kubernetes_endpoint_port_protocol`：端点端口的协议。
   *   `__meta_kubernetes_endpoint_address_target_kind`：端点地址目标的种类。
   *   `__meta_kubernetes_endpoint_address_target_name`：端点地址目标的名称。
*   如果端点属于服务，`role: service`则附加发现的所有标签。
*   对于由pod支持的所有目标，`role: pod`将附加发现的所有标签。

## **ingress**

该`ingress`角色发现了一个目标，为每个进入的每个路径。这通常用于黑盒监控入口。地址将设置为入口规范中指定的主机。

可用元标签：

*   `__meta_kubernetes_namespace`：入口对象的命名空间。
*   `__meta_kubernetes_ingress_name`：入口对象的名称。
*   `__meta_kubernetes_ingress_label_<labelname>`：入口对象中的每个标签。
*   `__meta_kubernetes_ingress_labelpresent_<labelname>`：`true`对于入口对象中的每个标签。
*   `__meta_kubernetes_ingress_annotation_<annotationname>`：来自入口对象的每个注释。
*   `__meta_kubernetes_ingress_annotationpresent_<annotationname>`：`true`对于来自入口对象的每个注释。
*   `__meta_kubernetes_ingress_scheme`：入口的协议方案，`https`如果设置了TLS配置。默认为`http`。
*   `__meta_kubernetes_ingress_path`：来自入口规范的路径。默认为`/`。
**有关Kubernetes发现的配置选项，请参见下文**：
```
# The information to access the Kubernetes API.

# The API server addresses. If left empty, Prometheus is assumed to run inside
# of the cluster and will discover API servers automatically and use the pod's
# CA certificate and bearer token file at /var/run/secrets/kubernetes.io/serviceaccount/.
# API服务器地址。 如果保留为空，则假定Prometheus在集群内部运行并自动发现API服务器，并在/var/run/secrets/kubernetes.io/serviceaccount/上使用pod的CA证书和不记名令牌文件。
[ api_server: <host> ]

# The Kubernetes role of entities that should be discovered.
# 应该被发现的实体的Kubernetes角色。
role: <role>

# Optional authentication information used to authenticate to the API server.
# Note that `basic_auth`, `bearer_token` and `bearer_token_file` options are
# mutually exclusive.
# password and password_file are mutually exclusive.
# 用于向API服务器进行身份验证的可选身份验证信息。请注意，`basic_auth`，`bearer_token`和`bearer_token_file`选项是互斥的.password和password_file是互斥的。

# Optional HTTP basic authentication information.
# 可选的HTTP基本认证信息。
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Optional bearer token authentication information.
# 可选的承载令牌认证信息。
[ bearer_token: <secret> ]

# Optional bearer token file authentication information.
# 可选的承载令牌文件认证信息。
[ bearer_token_file: <filename> ]

# Optional proxy URL.
# 可选的代理URL。
[ proxy_url: <string> ]

# TLS configuration.
tls_config:
  [ <tls_config> ]

# Optional namespace discovery. If omitted, all namespaces are used.
# 可选命名空间发现 如果省略，则使用所有名称空间。
namespaces:
  names:
    [ - <string> ]
```

有关为Kubernetes配置Prometheus的详细示例，请参阅[此示例Prometheus配置文件](https://github.com/prometheus/prometheus/blob/release-2.12/documentation/examples/prometheus-kubernetes.yml)。