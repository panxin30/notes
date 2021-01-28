### **参考文档**
[https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry](https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry) Concepts-->Containers-->Images 
最新版本`Nexus3.x`全面支持 Docker 的私有镜像。所以使用[`Nexus3.x`](https://www.sonatype.com/download-oss-sonatype/)一个软件来管理`Docker`,`Maven`,`Yum`,`PyPI`等是一个明智的选择。
```
docker run -d --name nexus3 --restart=always \
    -p 8081:8081 \     #web端口
	-p 8082:8082 \ #给私有仓库预留的端口，如果有
    --mount src=nexus-data,target=/nexus-data \
    sonatype/nexus3
```
### **连接私有仓库**
#### **方法1：**
其他机器需要连接仓库才能进行push、pull等操作
登录时，需要提供用户名和密码。认证的信息会被保存在~/.docker/config.json文件，在后续与私有镜像仓库交互时就可以被重用，而不需要每次都进行登录认证。
由于使用的是http协议，连接仓库前需要进行配置 vim /etc/docker/daemon.json
~~~
#在文件中添加如下的内容，告诉客户端这个私有镜像仓库是一个安全的仓库：
    {
    "insecure-registries": ["192.168.0.96:8082" ]
    }
    systemctl daemon-reload
    systemctl restart docker
~~~
docker info 确认下有没有成功添加私有仓库
```
...
Experimental: false
Insecure Registries:
 192.168.0.96:8082
 127.0.0.0/8
Live Restore Enabled: false
Product License: Community Engine
```
一但Node上有了这个配置，那么K8s就可以通过docker直接访问Private Registry了，这是[K8s文档中与私有镜像仓库交互的第一个方法](http://kubernetes.io/docs/user-guide/images/#using-a-private-registry)。考虑到Pod可以被调度到集群中的任意一个Node上，需要在每个Node上执行上述login操作，**或者可以简单地将**~/.docker/config.json scp到各个node上的~/.docker目录下。(在测试机上做测试的时候，k8s和gitlab，jenkins，仓库在一台服务器。使用login登录仓库留下的config.json还是k8s无法拉取到代码，又手动拷贝了一份到/var/lib/kubelete/config.json)
#### **方法2：通过kubectl创建docker-registry的secret**
```
Failed to pull image "192.168.0.96:8082/bw-custom:v7.10.1": rpc error: code = Unknown desc = Error 
response from daemon: Get http://192.168.0.96:8082/v2/bw-custom/manifests/v7.10.1:
no basic auth credentials
```
K8s提供的第二种方法是通过kubectl创建一个 docker-registry的secret，并在Pod描述文件中引用该secret以达到从Private Registry Pull Image的目的。

```
kubectl create secret docker-registry registrykey-m2-1 --docker-server=registry.cn-hangzhou.aliyuncs.com/xxxx/rbd-rest-api --docker-username={UserName} --docker-password={Password} --docker-email=team@domain.com
secret "registrykey-m2-1" created
kubectl get secret
 NAME TYPE DATA AGE
 registrykey-m2-1 kubernetes.io/dockercfg 1 29s
```
#### **方法3：通过secret yaml文件创建pull image所用的secret**
If you already ran`docker login`
cat pull-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: pull-secret
  namespace: tbw-pre
data:
    .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxOTIuMTY4LjAuOTY6ODA4MiI6IHsKCQkJImF1dGgiOiAiWVdSdGFXNDZUSGR2Y21zdVkyOXRNVEl6IgoJCX0KCX0sCgkiSHR0cEhlYWRlcnMiOiB7CgkJIlVzZXItQWdlbnQiOiAiRG9ja2VyLUNsaWVudC8xOC4wOS42IChsaW51eCkiCgl9Cn0=
type: kubernetes.io/dockerconfigjson
```
### `base64 -w 0 ~/.docker/config.json`
### **构建镜像及上传到仓库**
```
sudo docker login 172.16.77.71:8082 -u admin -p leanwork2018
sudo docker build . -t 172.16.77.71:8082/${ARTIFACT}:v${releaseNum}
sudo docker push 172.16.77.71:8082/${ARTIFACT}:v${releaseNum}
sudo docker rmi 172.16.77.71:8082/${ARTIFACT}:v${releaseNum}
sudo docker logout 172.16.77.71:8082
```