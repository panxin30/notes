+ rs replicaSet   kubectl explain rs
模板中定义的标签一定要符合标签选择器定义的
rs-demo.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
         name: myapp
         namespace: default
spec:
       replicas: 2
       selector:
              matchLabels:
                      app: myapp
                      release: canary
       template:
               metadata:
                        name: myapp-pod
                        labels:
                               app: myapp
                               release: canary
                               environment: qa
                spec:
                        container
                        - name: myapp-container
                          image: ikubernetes/myapp:v1
                          ports:
                          - name: http
                            containerPort: 80
```
