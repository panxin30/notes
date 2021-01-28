
+ **Pod生命周期中的重要行为**:
    初始化容器
    容器探测:
      liveness主容器是否处于运行状态
      readiness容器中的主进程是否已经准备就绪，并可以对外提供服务
**livenessProbe**：指示容器是否正在运行。如果存活探测失败，则 kubelet 会杀死容器，并且容器将受到其的影响。如果容器不提供存活探针，则默认状态为 Success。
**readinessProbe**：指示容器是否准备好服务请求。如果就绪探测失败，端点控制器将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址。初始延迟之前的就绪状态默认为 Failure。如果容器不提供就绪探针，则默认状态为 Success。
+ **探针类型**有三种：livenessProbe健康检测，readiness就绪检测，liftcycle?
**ExecAction**：在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。
**TCPSocketAction**：对指定端口上的容器的 IP 地址进行 TCP 检查。如果端口打开，则诊断被认为是成功的。
**HTTPGetAction**：对指定的端口和路径上的容器的 IP 地址执行 HTTP Get 请求。如果响应的状态码大于等于200 且小于 400，则诊断被认为是成功的。
### **该什么时候使用存活（liveness）和就绪（readiness）探针?**
如果容器中的进程能够在遇到问题或不健康的情况下自行崩溃，则不一定需要存活探针; kubelet 将根据 Pod 的restartPolicy 自动执行正确的操作。

如果您希望容器在探测失败时被杀死并重新启动，那么请指定一个存活探针，并指定restartPolicy 为 Always 或 OnFailure。

**如果要仅在探测成功时才开始向 Pod 发送流量，请指定就绪探针。在这种情况下，就绪探针可能与存活探针相同，但是 spec 中的就绪探针的存在意味着 Pod 将在没有接收到任何流量的情况下启动，并且只有在探针探测成功后才开始接收流量。**
在首次检查之前，初始状态为`Failure`
如果不配置，默认的状态为`Success`

如果您希望容器能够自行维护，您可以指定一个就绪探针，该探针检查与存活探针不同的端点。

请注意，如果您只想在 Pod 被删除时能够排除请求，则不一定需要使用就绪探针；在删除 Pod 时，Pod 会自动将自身置于未完成状态，无论就绪探针是否存在。当等待 Pod 中的容器停止时，Pod 仍处于未完成状态。
**例子**
```
        livenessProbe:
          failureThreshold: 3超时超过3次容器被标记为unready（这个pod的ip剔除出svc，不会有流量到这个ip）
          httpGet:
            path: /-/healthy
            port: 9090
            scheme: HTTP
          initialDelaySeconds: 30 容器启动30秒后启动探测
          periodSeconds: 10 探针每隔periodSeconds时间运行一次
          successThreshold: 1失败后探测成功的最小连续成功次数
          timeoutSeconds: 30探针在30秒后超时
# 对端口9090上的/-/healthy路径使用httpGet探测
#如果liveness检查失败，那么kubernetes将重新启动容器
        readinessProbe:
          failureThreshold: 3超时超过3次容器被标记为unready
          httpGet:
            path: /-/ready
            port: 9090
            scheme: HTTP
          initialDelaySeconds: 30 容器启动30秒后启动探测
          periodSeconds: 10 探针每隔periodSeconds时间运行一次
          successThreshold: 1失败后探测成功的最小连续成功次数
          timeoutSeconds: 30探针在30秒后超时
```
