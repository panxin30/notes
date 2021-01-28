## deployment-demo.yaml可以管理 Pod 的多个副本，并确保 Pod 按照期望的状态运行。
典型的用例如下：

- 使用Deployment来创建ReplicaSet。ReplicaSet在后台创建pod。检查启动状态，看它是成功还是失败。
- 然后，通过更新Deployment的PodTemplateSpec字段来声明Pod的新状态。这会创建一个新的ReplicaSet，Deployment会按照控制的速率将pod从旧的ReplicaSet移动到新的ReplicaSet中。
- 如果当前状态不稳定，回滚到之前的Deployment revision。每次回滚都会更新Deployment的revision。
- 扩容Deployment以满足更高的负载。
- 暂停Deployment来应用PodTemplateSpec的多个修复，然后恢复上线。
- 根据Deployment 的状态判断上线是否hang住了。
- 清除旧的不必要的 ReplicaSet。
kubectl explain deploy.spec.strategy.rollingUpdate{maxSurge,maxUnavailabe}更新策略，滚动更新,默认是各25%
如果只更新镜像 kubectl set image
kubectl explain deploy.spec.revisionHistoryLimit,滚动更新最多保留几个历史版本
```
deployment控制replicaset控制pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:（要匹配的标签为）
      app: myapp
      release: canary
  template:（使用模板来创建pod）
    metadata:
      labels:（一定要匹配上面标签选择器的标签）
        app: myapp
        release: canary
    spec:（pod的spec）
        containers：
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:
          - name: http
            containerPort: 80
```
